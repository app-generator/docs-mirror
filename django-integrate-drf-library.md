This page explains how to integrate the popular Django REST Framework (DRF) into an existing project. 

## What is DRF? 

First of all, what's an API? 

An API is just an interface that helps you communicate with another machine. 
On the web, an API is actually an interface you use to make your requests. 
Then this API will make what you want and send you a response. 
See it as a server in a restaurant. The backend is the kitchen and the frontend is the clients who want to eat.
Here the API is the server, simply because the server (API) receives requests (food ordering) from the clients (frontend) and then sends them to the kitchen (Backend). 
And once the meals (response) are ready, the server (API) will send these meals to the client to be consumed. (frontend)

DRF or Django Rest Framework is a python package built upon Django that helps you build REST API with Django backend. It comes with a lot of tools you can use to create your API, manage your endpoints, add authentication, and a lot more. 

## How to add an API to your Django project?  

First of all, you'll need to install the DRF package. 

```
pip install djangorestframework
```

Once it's done, add the `rest_framework` app in the `INSTALLED_APPS` list on the `settings.py` file of your project.

```
INSTALLED_APPS = [
    ...
    'rest_framework',
]
```

## Securing nodes

DRF allows you to secure your API by providing authentication classes in the `REST_FRAMEWORK` setting.

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',  
    ],
}
```

This setting for example is useful to implement a Token-based authentication.

## What is a serializer? 

A  [serializer](https://www.django-rest-framework.org/api-guide/serializers/)  allows you to convert complex Django data structures like `Queryset` or `Model` instance in Python native objects that can be easily converted to JSON/XML format. And it also works the inverse way, JSON/XML to Python native objects and then `Queryset` or `Model`.

For example, we have a model `Transaction`.

```python
from django.db import models


class Transaction(models.Model):
    product = models.CharField(max_length=50)
    price = models.IntegerField()
    qty = models.IntegerField(default=0)
    discount = models.IntegerField(default=0)
    info = models.CharField(max_length=250, null=True)
    created_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return f"{self.product}"
```

And then the serializer will look like this. 

```python
from api.transaction.models import Transaction
from rest_framework import serializers


class TransactionSerializer(serializers.ModelSerializer):
    info = serializers.CharField(allow_null=True, allow_blank=True)
    price = serializers.IntegerField(required=True)

    class Meta:
        model = Transaction
        fields = ["id", "product", "price", "qty", "discount", "info", "created_at"]
        read_only_field = ["id", "created_at"]
```

## How to test an API via POSTMAN?

Before testing your API with POSTMAN, we have to make sure to create views or viewsets. 

 [ViewSet](https://www.django-rest-framework.org/api-guide/viewsets/)  is a concept developed by DRF which consists of grouping a set of views for a given model in a single Python class. 
This set of views matches the predefined actions of CRUD type (Create, Read, Update, Delete), linked with HTTP methods.

```python
from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticated

from .models import Transaction
from .serializers import TransactionSerializer


class TransactionViewSet(viewsets.ModelViewSet):
    serializer_class = TransactionSerializer
    permission_classes = (IsAuthenticated,)
    http_method_names = ['get', 'post']

    def get_queryset(self):
        return Transaction.objects.all()
```

And then the last step to make this viewset reachable is to add it to a router. 

```python
from rest_framework import routers

from .viewsets import TransactionViewSet

router = routers.SimpleRouter(trailing_slash=False)

router.register('', TransactionViewSet, basename='transactions')

urlpatterns = [
    *router.urls,
]
```

And lastly, add the router configuration to the `urls.py` of the project. 

```python
from django.urls import path, include

urlpatterns = [
    path("api/transactions/", include(("api.routers.urls", "api-transactions"), namespace="api-transactions"))
]
```

`POST` and `GET` requests are working on this endpoint. 
Start the Django server. 

> **Get All transactions** - `http://localhost:8000/api/transactions`

```json
GET http://localhost:8000/api/transactions
```

> **Create a transaction** - `http://localhost:4000/api/transactions`

```json
POST http://localhost:4000/api/transactions
Content-Type: application/json

{
	"product": "Hot Dog",
	"qty": 1,
	"price": 10,
    "discount": 2,
    "info": "A simple hot dog"
}
```

