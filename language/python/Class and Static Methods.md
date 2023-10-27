
```python
class Person:
    def hello(arg='default'):
        print(f'Hello, with arg={arg}')

Person.hello()  # Hello, with arg=default

p = Person()
p.hello()   # Hello, with arg=<__main__.Person object at 0x10d1f7a00>
```



## Static Method

### Why Use Static Methods?

cases where it makes sense for a function to live in a class
but does not need access to either the instance or the class state

We sometimes also want to define functions in a class and always have them be just that - functions, never bound to either the class or the instance, however we call them. Often we do this because we need to utility function that is specific to our class, and we want to keep our class self-contained, or maybe we're writing a library of functions (though modules and packages may be more appropriate for this).

We can define static methods using the `@staticmethod` decorator: