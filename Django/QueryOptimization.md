# Main Takeaways:

- Table joins usually work best. We use that a lot all over. But when tables get very large (greater than hundreds of thousands of rows), subqueries seem to perform better. However sometimes they don’t perform great either, and that’s when using `get_limited_flat_query_list` is useful, so we can filter directly by id. This won’t work though when we hit some point, typically greater than 10 thousand ids returned.
    - Using `get_limited_flat_query_list` increases queries. If you got to this point though, it will likely speed things up more than if you were trying to reduce queries.
    - Sometimes using `get_limited_flat_query_list`, so try using subqueries first.
    - Check out The query optimization hierarchy:

# Specific Endpoint Takeaways

- Using subqueries vs. joins:
<img width="1000" alt="Screenshot 2024-08-25 at 8 48 54 AM" src="https://github.com/user-attachments/assets/6b5b62c6-3b20-45e9-8b91-4d3673b32d8e">

### Avoid queries in the serializer

Since the serializer is called once for each returned object, any queries triggered during serialization will run a large number of times, especially if the query is triggered by a related object which is not subject to a pagination limit.

```python
# theoretical MessageThreadSerializer:
# this will trigger one query for each serialized object
def get_visible_messages(self, obj):
    messages = obj.messages.filter(deleted=False, visible=True)
    return messages #ignore that these need to be serialized

# this will trigger an ungodly number of queries, guaranteed
def get_all_author_names(self, obj):
    messages = obj.messages.filter(deleted=False, visible=True)
    has_posted = set()
    for message in messages:
        has_posted.add(message.author.name)

In the above snippet there are two different issues. In the first, calling `.filter` inside the serializer will always be a query. If there are 5 threads being serialized, this will cause 5 queries. This is a small enough issue we may not even notice a performance issue, but should be removed nonetheless.

More significantly is the `get_all_author_names` function. Note that this function accesses properties of related objects, and if that object was not part of the original query it will need to be fetched individually. Also important is that even if, on the main queryset, you have prefetched:

```python
queryset.prefetch_related("messages__author")
```

since we are filtering messages, that prefetch is lost. That means that for each message a new query will be triggered to fetch the author AND while the threads themselves are limited by pagination (say no more than 20 threads per request), the messages are not. So our initial 1 query per serialized object can now be completely dwarfed by an uncapped number of queries. If this thread has 1000 messages, that's 1000 queries to get the authors for each of those messages. This is where we start to get some terrible performance.

There are a few ways to solve these issues. Starting with the simplest, we can modify our filter to include the `select_related` for author:

```python
def get_all_author_names(self, obj):
    messages = obj.messages.filter(deleted=False, visible=True).select_related("author")
    has_posted = set()
    for message in messages:
        has_posted.add(message.author.name)
```

This will take care of the uncapped queries, but still adds a query for the filter. In order to remove these queries, we need to go back to the base queryset.

```python
# before
queryset = Thread.objects.prefetch_related("messages__author")

# after
queryset = Thread.objects.prefetch_related(
    Prefetch(
        "messages",
        base_queryset=Messages.objects.filter(deleted=False, visible=True).select_related("author"),
        to_attr="visible_messages"
    ),
)
```

**Note:** The correct keyword argument for `Prefetch` should be `queryset` instead of `base_queryset`. Ensure to use the appropriate argument in your actual code.

This prefetch is much more specific; the initial queryset is performing the filter initially, then using the prefetch to associate the remaining messages with the correct threads all in one query.

```python
def get_visible_messages(self, obj):
    return obj.visible_messages # these would need to be serialized
```

```python
def get_all_author_names(self, obj):
    authors = set()
    for message in obj.visible_messages:
        authors.add(message.author.name)
```

This is only ideal if the number of messages we need to loop through is small, which is not likely to be the case here.

Instead of looping through values from the database to aggregate them, we can do it on the database. In this case, we assume we need the names as a comma-separated string, to match the changes made in the templates PR, but there are other ways to do aggregations.

```python
queryset = Thread.objects.all()....
    .annotate(author_names=StringAgg("message__author__name", delimiter=", ", distinct=True)
```

Now, as part of the initial query, the database will include a comma-separated list of the author names.

```python
def get_all_author_names(self, obj):
    return obj.author_names
```

With both these changes, we can now serialize these threads with no queries triggered during serialization.

Any function that returns a new queryset will cause a query in the serializer, so in addition to `.filter`, you should avoid using functions like `first`, `values_list`, and in some cases `exists` and `count`—instead, use `.all()[0]` in a try-except, `len`, and list comprehension.

### CRUDy Performance

In addition to reads, there are simple improvements to make in the creation, updating, and deleting of objects, in particular when operating on multiple objects at once.

Bulk creates and updates are the first option when creating an unknown number of objects at once. Typically, when creating and updating an object, you will use something like this.

```python
# create
form = Form.objects.create(**kwargs)

# update
form.response = []
form.save()

# delete
form.delete()
```

This works fine for creating a single object; it will trigger a single query to create an object and then return the new row, then again we can update the object and save it, triggering another query. Since each of these triggers a query, when dealing with more than one object, we are going to end up in the same situation where we are triggering one request per object.

Starting with bulk create, instead of using `Form.objects.create()`, we will construct the `Form` as a class in Python `Form(**kwargs)`. This creates an object with all the associated class functions but doesn’t send it to the database. We could trigger that by calling `.save()`, but instead we want to create a list of these, then create them all at once.

```python
forms_to_create = []
for user_id in users_to_assign_form:
    form = Form(**data, user_id=user_id)
    forms_to_create.append(form)
# creating - objects have no id
Form.objects.bulk_create(forms_to_create)
# updating - objects have an id
Form.objects.bulk_update(forms_to_create, ['user_id'])
```

Bulk update is the same, but the objects will need an ID, and you need to specify the fields to update.

For deleting and updating, there is another option which offers more flexibility: queryset update and delete.

```python
user_forms = Form.objects.filter(user=some_user, response__isnull=True)

# update all
user_forms.update(is_deleted=True)

# delete all
user_forms.delete()
```

For updates, you can also take advantage of database functions or aggregations, like `StringAgg` above, to calculate the new values in an update. Each of these can be executed in a single query rather than a series of individual queries.

### Specifics - Templates Improvements

As described, the changes here fit into one of: moving queries to a prefetch, filtering in Python, and aggregating on the DB.

```python
return [
    {
        "id": ownership.id,
        "user": ownership.user.id -> ownership.user_id
    } 
    for ownership in obj.ownerships.all()
]
```

In this case, the ownership was prefetched, but not the user. However, it's not necessary—the ID of the target object is stored on the ownership, and we can access it directly. By avoiding accessing properties of the related object, we prevent Django from needing to query the object.

```python
# before
return division.booking_pages.values_list("booking_uri", flat=True)
# after
return [
    booking_page.booking_uri for booking_page in division.booking_pages.all()
]
```

Unfortunately, while convenient, `values_list` triggers a new query. The simplest solution is to access the prefetched values and build the list yourself. It's possible, and `ArrayAgg` would be an even better solution.

```python
queryset = self.filter_queryset(self.get_queryset())
for ap in queryset:
    providers = [ownership.user for ownership in ap.ownerships.all()]
    writer.writerow(
        [
            ap.id,
            ap.name,
            ", ".join([str(provider.id) for provider in providers]),
            ", ".join([provider.name for provider in providers]),
            ", ".join([provider.email for provider in providers]),
            ap.division.id,
            ap.division.name,
            ap.is_draft,
        ]
    )

    organization = queryset.first().division.organization.name
```

Here is a very large aggregation. In this case, for each object in a queryset, we perform a series of simple loops to build comma-separated strings. Unfortunately, Python is way slower at these simple operations than Postgres. Secondarily, the `queryset.first()` will always trigger a query.

```python
queryset = self.filter_queryset(self.get_queryset()) \
    .select_related("division__organization") \
    .annotate(
        division_name=F("division__name"),
        providers_ids=StringAgg(Cast("ownerships__user_id", CharField()), ", "),
        providers_names=StringAgg(
            Concat(
                F("ownerships__user__first_name"),
                Value(" "),
                F("ownerships__user__last_name"),
            ),
            ", ",
        ),
        providers_emails=StringAgg("ownerships__user__email", ", "),
    )

for ap in queryset:
    writer.writerow(
        [
            ap.id,
            ap.name,
            ap.providers_ids,
            ap.providers_names,
            ap.providers_emails,
            ap.division_id,
            ap.division_name,
            ap.is_draft,
        ]
    )

    try:
        organization = queryset[0].division.organization.name
    except IndexError:
        organization = ""
```

We move all these aggregations to the DB, and it's able to fly through them. We also replace the `first()` with a 0 index since we have already evaluated this query.

```python
# before
new_intake_forms = [
    AppointmentProductIntakeForm.objects.get_or_create(
        product=instance, form=patient_form
    )[0]
    for patient_form in patient_forms
]
instance.intakes.exclude(form_id__in=form_ids).delete()
instance.intakes.set(new_intake_forms)

# after
new_intake_forms = [
    AppointmentProductIntakeForm(
        product_id=instance.id, form_id=patient_form.id
    )
    for patient_form in patient_forms
]
# _we expect the number of intakes to be low so this is not too wasteful
instance.intakes.all().delete()
AppointmentProductIntakeForm.objects.bulk_create(new_intake_forms)
```

Where before the code was looking for an existing object before creating, we now instead just delete all the existing ones. Since intakes generally have a low count and this is not a common operation, the extra cost is low. Then we can take advantage of `bulk_create` to create an entirely new set.

A similar change is made for the ownerships, but since there are more, we maintain the logic of filtering, but do it through prefetched data and then `bulk_create` the new ownerships.

# Conceptual Takeaways

## The query optimization hierarchy:

1. **Use joins first:** 

    ```python
    participant_forms = (
        PatientFormAssignment.objects
        .annotate_has_unfulfilled_client_signatures()
        .filter(
            participant__cancelled=False,
            participant__email__iexact=obj.email,
            participant__appointment__cancellation=None,
        )
        .filter(
            Q(response__isnull=True) | Q(has_unfulfilled_client_signatures=True)
        )
    )
    ```

2. **If that is not performing well, force a subquery:**

    ```python
    appointment_participants = AppointmentParticipant.objects.filter(
        cancelled=False,
        email__iexact=obj.email,
        appointment__cancellation=None
    )
    participant_forms = (
        PatientFormAssignment.objects
        .annotate_has_unfulfilled_client_signatures()
        .filter(
            participants__in=appointment_participants
        )
        .filter(
            Q(response__isnull=True) | Q(has_unfulfilled_client_signatures=True)
        )
    )
    ```

3. **If this is still not performing well, try using `get_limited_flat_query_list`:**

    ```python
    participants_query = Q(
        cancelled=False,
        email__iexact=obj.email,
        appointment__cancellation=None,
    ) & Q(
        id__in=PatientFormAssignment.objects.filter(
            participant_id__isnull=False
        ).values_list("participant_id", flat=True)
    )
    participant_ids = get_limited_flat_query_list(
        AppointmentParticipant.objects.filter(participants_query),
        bugsnag_meta={"user_id": obj.id},
    )
    participant_forms = (
        PatientFormAssignment.objects
        .annotate_has_unfulfilled_client_signatures()
        .filter(participant_id__in=participant_ids)
        .filter(
            Q(response__isnull=True) | Q(has_unfulfilled_client_signatures=True)
        )
    )
    ```

In this example, #3 performed the best. Although it takes 2 queries instead of 1, this takes under 15ms total…!

We want to reduce queries where possible, but sometimes it’s worth having 1 extra query that reduces total response time by a lot. 

For example, here is an example of an endpoint that is suffering from the N+1 query issue, and it means that most of the response time comes from Django instead of the DB:

This is due to the DB connection taking some time to setup and tear down, multiplied by the hundreds of extra queries happening here.

## `prefetch_utils.SerializerMeta`

This thing is awesome and helps with prefetching objects for serializers.

- Define prefetches in the `prefetch_related` list. Each attribute and nested attribute here will result in one query.
- Define `select_related` prefetches as well using the `select_related` list. Each attribute here **will also result in one query (including nested attributes).** If you were to define `.select_related()` on the view set queryset, however, that would result in table joining happening on the main query. **This means no new queries.**

It’s important to know what results in a table join, and what results in a query.

Table joins do not mean faster response times everytime.

An example PR where I attempted to balance joins vs. prefetching in the `PatientFormAssignmentSerializer` class: https://github.com/oncallhealth/OnCall/pull/5351/files

(This may or may not have worked ultimately; I noticed the response time was very similar before/after, but queries to `/api/patientformassignments/` were definitely reduced.)
```
