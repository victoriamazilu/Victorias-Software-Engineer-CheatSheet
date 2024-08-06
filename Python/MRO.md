# Python Method Resolution Order (MRO) Notes

## Defintion

- **Method Resolution Order (MRO)** defines the order in which methods are looked up in a hierarchy of classes.
- Essential in multiple inheritance to determine which method to invoke.

## Single Inheritance

- MRO starts from the class itself and moves up the hierarchy.
- Example:

    ```python
    class A:
        def method(self):
            print("A")

    class B(A):
        pass

    b = B()
    b.method()  # Output: A
    ```

## Multiple Inheritance

- Python uses the **C3 Linearization Algorithm** to determine the MRO.
- Ensures a consistent method lookup order.
- Example:

    ```python
    class A:
        def method(self):
            print("A")

    class B(A):
        def method(self):
            print("B")

    class C(A):
        def method(self):
            print("C")

    class D(B, C):
        pass

    print(D.mro())
    # Output: [D, B, C, A, object]
    ```

## Using `super()`

- `super()` function uses MRO to determine which method to call.
- Useful in cooperative multiple inheritance.
- Example:

    ```python
    class A:
        def method(self):
            print("A")

    class B(A):
        def method(self):
            print("B")
            super().method()

    class C(A):
        def method(self):
            print("C")
            super().method()

    class D(B, C):
        def method(self):
            print("D")
            super().method()

    d = D()
    d.method()
    # Output: D, B, C, A
    ```

## Practical Example in Django

- MRO is used in frameworks like Django for class-based views.
- Example:

    ```python
    from django.views import View

    class MyBaseView(View):
        def get(self, request, *args, **kwargs):
            print("Base GET")

    class MyMixin:
        def get(self, request, *args, **kwargs):
            print("Mixin GET")
            return super().get(request, *args, **kwargs)

    class MyView(MyMixin, MyBaseView):
        def get(self, request, *args, **kwargs):
            print("MyView GET")
            return super().get(request, *args, **kwargs)

    # MRO: MyView -> MyMixin -> MyBaseView -> View -> object
    ```

## Summary

1. **MRO** ensures consistent method lookup in multiple inheritance.
2. **Use `mro()`** to inspect the MRO of a class.
3. **`super()`** leverages MRO for method calls from parent classes.
4. 
