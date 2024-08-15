### Django Signals Cheat Sheet

#### What are Django Signals?
Django signals are a way to allow certain senders to notify a set of receivers when some action has taken place. They provide a mechanism to decouple the code that triggers an event from the code that handles the event, allowing for cleaner, more modular code.

#### When to Use Django Signals?
- **Post-Save Actions**: When you need to perform an action after a model instance is saved.
- **Pre-Save Actions**: When you need to modify data before it is saved to the database.
- **Post-Delete Actions**: When you need to clean up related data after a model instance is deleted.
- **Custom Events**: When you want to trigger actions based on custom business logic.

#### Key Components
- **Sender**: The model or event that triggers the signal.
- **Receiver**: The function that gets executed when the signal is triggered.
- **Signal**: The event that gets emitted.

#### Commonly Used Signals
1. **post_save**: Triggered after a model's `save()` method is called.
2. **pre_save**: Triggered before a model's `save()` method is called.
3. **post_delete**: Triggered after a model instance is deleted.
4. **pre_delete**: Triggered before a model instance is deleted.

#### How to Connect Signals
There are two main ways to connect signals in Django:
1. **Using the `@receiver` decorator**:
   ```python
   from django.db.models.signals import post_save
   from django.dispatch import receiver

   @receiver(post_save, sender=MyModel)
   def my_handler(sender, instance, **kwargs):
       # Logic to execute after saving MyModel instance
   ```

2. **Using `signal.connect()`**:
   ```python
   from django.db.models.signals import post_save

   def my_handler(sender, instance, **kwargs):
       # Logic to execute after saving MyModel instance

   post_save.connect(my_handler, sender=MyModel)
   ```

#### Example: Post-Save Signal
```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from myapp.models import MyModel

@receiver(post_save, sender=MyModel)
def my_handler(sender, instance, **kwargs):
    # Example: Send a confirmation email after saving a model instance
    send_confirmation_email(instance.user.email)
```

#### Signal Arguments
- **sender**: The model class that sent the signal.
- **instance**: The actual instance of the model that was saved or deleted.
- **created**: A boolean; `True` if a new record was created, `False` if an existing record was updated.
- **kwargs**: Additional keyword arguments.

#### Tips for Using Django Signals
- **Decouple Logic**: Use signals to separate concerns. For instance, handling side effects (like sending emails) outside the main business logic.
- **Be Cautious with Performance**: Signals can introduce performance overhead, especially if they trigger heavy tasks.
- **Avoid Overuse**: While signals are powerful, overusing them can make the codebase harder to understand and maintain. Use them judiciously.
- **Debugging**: Use `print` statements or logging inside your signal handlers to help debug issues.

#### Testing Signals
- Ensure your signals are working as expected by writing unit tests for them. You can use Django’s `override_settings` or `@mock.patch` to test the behavior of signals without side effects.

#### Disconnecting Signals
If needed, you can disconnect a signal using:
```python
from django.db.models.signals import post_save

post_save.disconnect(my_handler, sender=MyModel)
```
