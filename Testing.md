# Parameterizing Tests

Parameterized tests allow you to run the same test logic with different inputs, reducing code duplication and making your tests easier to manage and extend. Here's a concise guide on how to use parameterized tests, with an example:

### Benefits of Parameterized Tests

- **Reduce Code Duplication**: Instead of writing multiple test functions with similar logic, you can write one test function and run it with various inputs.
- **Improve Test Coverage**: Easily test edge cases and a variety of input values.
- **Maintainability**: Easier to update tests since changes need to be made in one place.

### Setting Up Parameterized Tests

#### Install parameterized Package

If you haven’t already, install the parameterized package:

```sh
pip install parameterized
```

#### Import the parameterized Decorator

```python
from parameterized import parameterized
```

#### Define Parameterized Test

Use the `@parameterized.expand` decorator to specify the different sets of parameters for your test.

### Example

Below is an example of how to refactor two similar test cases into a single parameterized test:

#### Original Tests
Note that the two tests are identical except for updated_data -> notes

```python
def test_note_empty_succeeds(self):
    division = create_division({})
    provider, provider_password = create_test_provider(division=division)
    contact = create_contact(
        custom_attributes={
            "provider": provider,
            "division_id": division.id,
            "notes": "Initial note",
        }
    )
    self.contact_url = reverse("contact-detail", args=[contact.id])
    updated_data = {
        "provider": provider.id,
        "division_id": division.id,
        "first_name": contact.first_name,
        "last_name": contact.last_name,
        "email": contact.email,
        "phone": contact.phone,
        "notes": "",
    }
    self.client.login(email=provider.email, password=provider_password)
    response = self.client.put(self.contact_url, updated_data)
    self.assertEqual(response.status_code, status.HTTP_200_OK)
    contact.refresh_from_db()
    if test_notes is None:
        self.assertIsNone(contact.notes)
    else:
        self.assertEqual(contact.notes, "")

def test_note_null_succeeds(self):
    division = create_division({})
    provider, provider_password = create_test_provider(division=division)
    contact = create_contact(
        custom_attributes={
            "provider": provider,
            "division_id": division.id,
            "notes": "Initial note",
        }
    )
    self.contact_url = reverse("contact-detail", args=[contact.id])
    updated_data = {
        "provider": provider.id,
        "division_id": division.id,
        "first_name": contact.first_name,
        "last_name": contact.last_name,
        "email": contact.email,
        "phone": contact.phone,
        "notes": None,
    }
    self.client.login(email=provider.email, password=provider_password)
    response = self.client.put(self.contact_url, updated_data)
    self.assertEqual(response.status_code, status.HTTP_200_OK)
    contact.refresh_from_db()
    if test_notes is None:
        self.assertIsNone(contact.notes)
    else:
        self.assertEqual(contact.notes, None)
```

#### Parameterized Test

```python
from parameterized import parameterized
from django.urls import reverse
from rest_framework import status

class ContactTests(TestCase):

    @parameterized.expand([
        ("empty_notes", ""),
        ("null_notes", None),
    ])
    def test_note_value_succeeds(self, name, test_notes):
        division = create_division({})
        provider, provider_password = create_test_provider(division=division)
        contact = create_contact(
            custom_attributes={
                "provider": provider,
                "division_id": division.id,
                "notes": "Initial note",
            }
        )
        self.contact_url = reverse("contact-detail", args=[contact.id])
        updated_data = {
            "provider": provider.id,
            "division_id": division.id,
            "first_name": contact.first_name,
            "last_name": contact.last_name,
            "email": contact.email,
            "phone": contact.phone,
            "notes": test_notes,
        }
        self.client.login(email=provider.email, password=provider_password)
        response = self.client.put(self.contact_url, updated_data)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        contact.refresh_from_db()
        if test_notes is None:
            self.assertIsNone(contact.notes)
        else:
            self.assertEqual(contact.notes, test_notes)
```

### Explanation

1. **Import parameterized**: Import the `parameterized` decorator from the `parameterized` package.
2. **Define Parameterized Test**: Use `@parameterized.expand` to specify different inputs. Here, it tests both an empty string (`""`) and `None` for the notes field.
3. **Single Test Function**: The test function `test_note_value_succeeds` takes `name` and `test_notes` as parameters. The test logic is executed for each set of inputs defined in the `@parameterized.expand` decorator.
4. **Assertions**: The test checks the response status and ensures the notes field in the contact is updated correctly based on the input.

By using parameterized tests, you can streamline your testing process, ensuring that your tests are more concise and easier to maintain.
