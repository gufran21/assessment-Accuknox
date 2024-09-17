# Assessment for Django Trainee at Accuknox

## Topic: Django Signals

**Question 1:** By default, are Django signals executed synchronously or asynchronously? Please support your answer with a code snippet that conclusively proves your stance. The code does not need to be elegant and production ready, we just need to understand your logic.

**Answer 1:** By default, **Django signals are executed synchronously**. This means that when a signal is sent, the associated signal handlers run immediately in the same thread and block further execution until they are finished.

To demonstrate this, I will provide a simple example using Django’s `post_save` signal. In the code, we will:

- Save a model instance.
- Log timestamps before and after saving the model.
- Inside the signal handler, log when the handler starts and ends, simulating a delay to make the synchronous nature clear.



#### Code Example:

```python
import time
from django.db import models
from django.db.models.signals import post_save
from django.dispatch import receiver

# Define a simple model
class MyModel(models.Model):
    name = models.CharField(max_length=100)

# Signal handler for post_save
@receiver(post_save, sender=MyModel)
def my_model_post_save_handler(sender, instance, **kwargs):
    # Log current time in the signal handler
    print(f"Signal handler started at: {time.time()}")
    time.sleep(2)  # Simulate delay
    print(f"Signal handler ended at: {time.time()}")

# Function to save a model instance and check the time
def save_model_instance():
    print(f"Saving model instance at: {time.time()}")
    instance = MyModel(name="Test")
    instance.save()
    print(f"Model instance saved at: {time.time()}")

# Call the function
save_model_instance()
```

#### Explanation:
- The `save()` function logs the start time, creates and saves the model instance.
- Immediately after `save()` is called, the `post_save` signal is triggered, and the signal handler logs the start and end time.
- The `save_model_instance()` function only completes after the signal handler has finished running, which proves that Django signals are executed **synchronously** by default.



**Question 2:** Do django signals run in the same thread as the caller? Please support your answer with a code snippet that conclusively proves your stance. The code does not need to be elegant and production ready, we just need to understand your logic.

**Answer 2:** Yes, **Django signals run in the same thread as the caller**. This means that when a signal is emitted, the connected signal handlers are executed in the same thread that triggered the signal.

To conclusively prove this, we can:
- Save a model instance in Django.
- Check the thread ID before the `save()` call.
- In the signal handler, check the thread ID again.
Both thread IDs should be the same, indicating that the signal handler runs in the same thread as the caller.

#### Code Example:

```python

import time
import threading
from django.db import models
from django.db.models.signals import post_save
from django.dispatch import receiver

# Define a simple model
class MyModel(models.Model):
    name = models.CharField(max_length=100)

# Signal handler for post_save
@receiver(post_save, sender=MyModel)
def my_model_post_save_handler(sender, instance, **kwargs):
    # Log the thread ID in the signal handler
    print(f"Signal handler thread ID: {threading.get_ident()}")

# Function to save a model instance and check the thread ID
def save_model_instance():
    print(f"Caller thread ID: {threading.get_ident()}")
    instance = MyModel(name="Test")
    instance.save()

# Call the function
save_model_instance()

```

#### Explanation:
- The `save_model_instance()` function logs the thread ID before the `save()` call.
- When the `post_save` signal is triggered, the signal handler logs the thread ID again.
- Both thread IDs are the same, proving that Django signals run in the **same thread** as the caller.

**Question 3:** By default, do Django signals run in the same database transaction as the caller?


**Answer 3:** Yes, **Django signals run in the same database transaction as the caller by default**. This means that if a signal is triggered inside a database transaction, the signal handlers are executed within the same transaction context.

To conclusively prove this, we can:
- Use a Django model and `post_save` signal.
- Open a database transaction, save a model instance, and deliberately raise an exception after the save to force a transaction rollback.
- Check if the signal handler's database changes are also rolled back when the transaction is rolled back.



#### Code Example:
```python
from django.db import models, transaction
from django.db.models.signals import post_save
from django.dispatch import receiver

# Define a simple model
class MyModel(models.Model):
    name = models.CharField(max_length=100)

# A separate model to track signal handler changes
class SignalLog(models.Model):
    message = models.CharField(max_length=100)

# Signal handler for post_save
@receiver(post_save, sender=MyModel)
def my_model_post_save_handler(sender, instance, **kwargs):
    print("Signal handler running")
    # Create a log entry to check if it's saved or rolled back
    SignalLog.objects.create(message="Signal handler executed")

# Function to save a model instance and force a transaction rollback
def save_model_with_transaction():
    try:
        with transaction.atomic():
            # Save model instance inside a transaction
            instance = MyModel(name="Test")
            instance.save()
            print("Model saved, now raising exception...")
            # Raise an exception to force a rollback
            raise Exception("Forcing rollback")
    except Exception as e:
        print(f"Exception caught: {e}")

# Call the function
save_model_with_transaction()

# Check if SignalLog contains any entries
print(f"SignalLog count: {SignalLog.objects.count()}")
```

#### Explanation:
- The `save_model_with_transaction()` function saves the `MyModel` instance inside a transaction.
- The `post_save` signal is triggered, and the signal handler creates an entry in the `SignalLog` model.
- An exception is raised after saving, causing the entire transaction (including the signal handler's changes) to be rolled back.
- After the exception, we check if any `SignalLog` entries were saved. Since the count is 0, it proves that the signal handler's changes were rolled back along with the transaction.



## Topic: Custom Classes in Python

**Description:** You are tasked with creating a `Rectangle` class with the following requirements:
1. An instance of the `Rectangle` class requires `length: int` and `width: int` to be initialized.

2. We can iterate over an instance of the `Rectangle` class.

3. When an instance of the `Rectangle` class is iterated over, we first get its length in the format: `{'length': <VALUE_OF_LENGTH>}` followed by the width in the format: `{'width': <VALUE_OF_WIDTH>}`.


#### Code:

```python
class Rectangle:
    def __init__(self, length: int, width: int):
        # Initialize the rectangle with length and width
        self.length = length
        self.width = width

    def __iter__(self):
        # Return an iterator that yields the length and width as dictionaries
        yield {'length': self.length}
        yield {'width': self.width}

# Example usage
rectangle = Rectangle(10, 5)

for dimension in rectangle:
    print(dimension)


```

**Explanation:**

- The `Rectangle` class takes `length` and `width` as required parameters.

- The `__iter__` method is defined to make the instance iterable. It uses Python's `yield` to return the length and width in the specified format.

- When the `Rectangle` instance is iterated over, the `__iter__` method is called, and it returns the length and width as dictionaries in the order specified.





