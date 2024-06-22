# Django REST Framework Notes

## Models

-   **Models** define the structure of your database. They are Python classes that subclass `django.db.models.Model`.
-   Each attribute in a model represents a database field.
-   Use various field types like `CharField`, `IntegerField`, `DateField`, etc.
-   Example:

    ```python
    from django.db import models

    class Book(models.Model):
        title = models.CharField(max_length=100)
        author = models.CharField(max_length=100)
        published_date = models.DateField()
    ```

## Serializers

-   **Serializers** convert complex data types (like model instances) to native Python datatypes that can be easily rendered into JSON, XML, or other content types.
-   They also handle deserialization, converting parsed data back into complex types.
-   Example:

    ```python
    from rest_framework import serializers
    from .models import Book

    class BookSerializer(serializers.ModelSerializer):
        class Meta:
            model = Book
            fields = '__all__'
    ```

## Views

-   **Views** handle the logic of your application, processing requests and returning responses.
-   DRF provides two types of views: `APIView` and ViewSets.
    -   `APIView` gives more control but requires more code.
    -   **ViewSets** provide more abstraction and less boilerplate.
-   Example using a ViewSet:

    ```python
    from rest_framework import viewsets
    from .models import Book
    from .serializers import BookSerializer

    class BookViewSet(viewsets.ModelViewSet):
        queryset = Book.objects.all()
        serializer_class = BookSerializer
    ```

## URLs

-   **urls.py** is where you route your views to specific URLs.
-   You can use DRF's `DefaultRouter` to automatically generate URL patterns for your ViewSets.
-   Example:

    ```python
    from django.urls import path, include
    from rest_framework.routers import DefaultRouter
    from .views import BookViewSet

    router = DefaultRouter()
    router.register(r'books', BookViewSet)

    urlpatterns = [
        path('', include(router.urls)),
    ]
    ```

## Flow in General

1. **Define Models**: Create your data structures in `models.py`.
2. **Create Serializers**: In `serializers.py`, create serializers to convert your models to/from JSON.
3. **Write Views**: In `views.py`, create your views or viewsets to handle request logic.
4. **Configure URLs**: In `urls.py`, route URLs to your views using DRF's router.
5. **Test Endpoints**: Use tools like Postman or DRF's built-in API browser to test your API endpoints.
