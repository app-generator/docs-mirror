This page explains how to extend the default User Model in Django and associate with a user other information like address, phone number .. etc.

Django provides `User` models you can use for authentication or identification. But what if you need more? Like adding more fields such as addresses, phone or city ... etc.

You'll need to extend the `User` model. Most of the time, it consists of two steps: 
- Extend the model using `AbstractUser`
- Create a new manager the new model created

## Extending the User model

For this, we'll create a new class with new fields. 

```python
class CustomUser(AbstractUser):
    """
    The User model has already:
    - first name
    - last name
    - email
    - username
    - is_active
    - is_staff
    - date_joined
    - last_login
    """

    phone = models.CharField(max_length=35, blank=True, null=True)
    address = models.CharField(max_length=255, blank=True, null=True)
    city = models.CharField(max_length=35, blank=True, null=True)
```

## Creating a custom manager

The model manager tells Django how to manage the models created under this manager. It can be methods such as `created_user` or `create_superuser`. We'll be rewriting these methods. 

```python
from django.contrib.auth.models import AbstractUser, BaseUserManager
from django.conf import settings
from django.utils.translation import ugettext_lazy as _


class CustomManager(BaseUserManager):

    def create_user(self, email, username, password, **extra_fields):
        """
        Create and save a User with the given username, email, and password.
        """
        if not email:
            raise ValueError(_('The Email must be set'))
        email = self.normalize_email(email)
        user = self.model(email=email, username=username, **extra_fields)
        user.set_password(password)
        user.save()
        return user

    def create_superuser(self, email, username, password, **extra_fields):
        """
        Create and save a SuperUser with the given username, email, and password.
        """
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)
        extra_fields.setdefault('is_active', True)

        if extra_fields.get('is_staff') is not True:
            raise ValueError(_('Superuser must have is_staff=True.'))
        if extra_fields.get('is_superuser') is not True:
            raise ValueError(_('Superuser must have is_superuser=True.'))
        return self.create_user(email=email, password=password, username=username, **extra_fields)
```

The final code will look like this. 

```python
from django.db import models
from django.contrib.auth.models import AbstractUser, BaseUserManager
from django.utils.translation import ugettext_lazy as _


class CustomUserManager(BaseUserManager):

    def create_user(self, email, username, password, **extra_fields):
        """
        Create and save a User with the given username, email, and password.
        """
        if not email:
            raise ValueError(_('The Email must be set'))
        email = self.normalize_email(email)
        user = self.model(email=email, username=username, **extra_fields)
        user.set_password(password)
        user.save()
        return user

    def create_superuser(self, email, username, password, **extra_fields):
        """
        Create and save a SuperUser with the given username, email, and password.
        """
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)
        extra_fields.setdefault('is_active', True)

        if extra_fields.get('is_staff') is not True:
            raise ValueError(_('Superuser must have is_staff=True.'))
        if extra_fields.get('is_superuser') is not True:
            raise ValueError(_('Superuser must have is_superuser=True.'))
        return self.create_user(email=email, password=password, username=username, **extra_fields)


class CustomUser(AbstractUser):
    """
    The User model has already:
    - first name
    - last name
    - email
    - username
    - is_active
    - is_staff
    - date_joined
    - last_login
    """

    phone = models.CharField(max_length=35, blank=True, null=True)
    address = models.CharField(max_length=255, blank=True, null=True)
    city = models.CharField(max_length=35, blank=True, null=True)

    objects = CustomUserManager()
```

And lastly, you need to tell Django that it should use your new `User` model.

In the `settings.py` file of your project, add this line of code: 

```python
AUTH_USER_MODEL = 'CustomUser'
```

You can check an example of a template code just  [here](https://github.com/app-generator/boilerplate-code-django-dashboard/blob/master/apps/profile/models.py). 
