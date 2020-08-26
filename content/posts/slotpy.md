+++
title = "What is __slots__ in Python?"
description = "An introduction to the use of `__slots__`"
tags = [
    "python",
    "development",
]
date = 2020-08-26T08:13:50Z
author = "Lorenzo Peppoloni"
+++

## tl;dr
Every Python class can have instance attributes that can be dynamically added/removed/modified. This increases memory usage and results in slower attributes access. If you need to optimize, you can avoid dynamic attributes creation by defining `__slots__`. Python will now instantiate a static amount of memory to only contain the specified attributes.

## Python classes attributes

In Python every class can have instance attributes. This attributes by default are stored in a dict. This has the advantage of being able to dynamically add attributes to a class, for example you can do:

```py
class Foo():
    def __init__(self):
        self.a = 10

f1 = Foo()
f1.b = 20
```

In this case the attribute `b` is added dynamically to the class instance `f`. 

If we inspect the attributes of the object by using `dir()` we can see `__dict__`, which is the dictionary containing the attributes of the instance. 

```py
print(f.__dict__)
{'a': 10, 'b': 20}
```

Note that this cannot be done with built-in classes, for example:

```py
arr = numpy.ones(10)
arr.foo = 10

num = 10
num.foo = 2

one_set = set([12, 13])
one_set.foo = 11
```

they will all raise an `AttributeError` exception.

## No dynamic attributes

If we define `__slots__` with a list of attributes, we will prevent the dynamic creation of attributes for the class. Let's modify the first class we created:

```py
class Foo():
    __slots__ = ["a"]

    def __init__(self):
        self.a = 10

f = Foo()
f.b = 20
```
Now we get `AttributeError: 'Foo' object has no attribute 'b'`.

If we inspect the attributes of the object, we will see that there is no `__dict__` anymore, but we have `__slots__` containing the list of attributes, in this case `["a"]`.

### Inheritance

When using inheritance, if the base class has `__slots__` defined, it will pass it down the inheritance tree, so there will be no need to re-define it for the inherited attributes. Note that Python does not complain, but you will be using more memory than expected.

```py
class Base:
    __slots__ = ["a", "b"]

class Foo(Base):
    __slots__ = ["c"]  # Correct: Foo already has ["a", "b"] inherited, thus having ["a", "b", "c"]

class Bar(Base):
    __slots__ = ["a", "b", "c"]  # Wrong: no need to re-define ["a", "b"]
```
By using `getsizeof` we can see that:

```bash
>>> sys.getsizeof(Foo())
72
>>> sys.getsizeof(Bar())
88
```

## Why to use `__slots__`?

There are two main reasons:

1) Faster access to attributes

This is the actual reason why `__slots__` was introduced. Quoting the [History of Python](http://python-history.blogspot.com/2010/06/inside-story-on-new-style-classes.html) blog

> Some people mistakenly assume that the intended purpose of `__slots__` is to increase code safety (by restricting the attribute names). In reality, my ultimate goal was performance. 


2) Less used memory

In general the default `dict` uses a lot of memory, because we cannot just allocate a static amount of memory for the class instance. This can take a toll when we create thousands or millions of objects. By using `__slots__` Python will only allocate space for the specified set of attributes.

## Further reading

[Official Documentation](https://docs.python.org/3/reference/datamodel.html#slots)

https://stackoverflow.com/questions/472000/usage-of-slots

