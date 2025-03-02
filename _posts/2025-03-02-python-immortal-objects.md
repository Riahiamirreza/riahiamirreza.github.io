# Immortal objects in Python

People often call `tuple` or `str` in Python _immutable_, but it's not really true. Not because you can REALLY mutate them like below:
```python
Python 3.13.1 (main, Dec  3 2024, 17:59:52) [Clang 16.0.0 (clang-1600.0.26.4)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import ctypes
>>> 
>>> t = ("hello", 1, 2, 3)
>>> t_size = ctypes.c_long.from_address(id(t) + ctypes.sizeof(ctypes.c_long) * 2)
>>> t_size
c_long(4)
>>> 
>>> t_size.value = 3
>>> 
>>> t
('hello', 1, 2)
>>> t_size.value = 2
>>> t
('hello', 1)
>>> t_size.value = 0
>>> t
()
>>> t_size.value = 2
>>> t
('hello', 1)
>>> t_size.value = 5
>>> t
('hello', 1, 2, 3, <NULL>)
>>>
```
But because it is only mutable from the perspective of the user, the user of that data type can assume that they can not be mutated. But from the memory model perspective, which is very important when dealing with multi-threading, immutable objects, were not truly immutable, because their reference count changes.
```python
>>> import sys
>>> 
>>> t = (2, 4, "somestring")
>>> sys.getrefcount(t)
2
>>> list_of_t = [t for _ in range(1000)]
>>> sys.getrefcount(t)
1002
```

As you can see, the reference count of an object (even _immutable_ ones) can change, so the object is not immutable in terms of bytes stored in the memory.
## Immortal Objects
Having truly immutable objects can make us happier when writing multi-threaded program. If an object is immutable, then multiple threads can access it at the same time without any mutex. Immortal objects which are introduced in [PEP-683](https://peps.python.org/pep-0683/) are objects which has fixed reference count number. This property make them stay alive during whole execution time. Because the reference count is fixed (and non-zero), it never reaches zero, thus never get deallocated.
The below, is REPL for Python 3.10:
```python
Python 3.10.16 (main, Dec  3 2024, 17:27:57) [Clang 16.0.0 (clang-1600.0.26.4)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> 
>>> sys.getrefcount(None)
4882
>>> 
>>> list_of_nones = [None for _ in range(10_000)]
>>> sys.getrefcount(None)
14880
```
As you can see, reference count of `None` changes when I create a list of `None`s. Now compare that to the below, which is for Python 3.13:
```python
Python 3.13.1 (main, Dec  3 2024, 17:59:52) [Clang 16.0.0 (clang-1600.0.26.4)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.getrefcount(None)
4294967295
>>> list_of_nones = [None for _ in range(10_000)]
>>> sys.getrefcount(None)
4294967295
```
The reference count of `None` is fixed (and very very large) number, and it does not increase when I create a list of `None`.
Making some objects immortal make sense, such as `None`, `True` and `False` as we're pretty sure that we want them to stay alive during the execution.
