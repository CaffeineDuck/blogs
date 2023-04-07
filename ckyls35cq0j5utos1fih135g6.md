---
title: "How to convert the type of function's arguments in Python at runtime?"
seoDescription: "Converting the type of arguments passed to the function according to the type hints of the parameters of a function."
datePublished: Wed Jan 19 2022 16:47:26 GMT+0000 (Coordinated Universal Time)
cuid: ckyls35cq0j5utos1fih135g6
slug: how-to-convert-the-type-of-functions-arguments-in-python-at-runtime
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1642611813493/rkAcQWSFj.png
tags: functions, python

---

# Introduction

Most of us have run into a situation where we have to change the type of a parameter in function; either it is simple `str` -> `int` or something far more complex. But at some point, we'll be repeating the code too much just for converting the types of the parameter instead of actual logic which drastically increases the code length and decreases readability. 

Example:
```py
# Defining a function with Type Hints (I'll explain this below)
def greater(a: int, b: int) -> int:
    # Converting from str to int
    a = int(a)
    b = int(b)
    # This is equivalent of 
    # if a > b:
    #     return a
    # else:
    #     return b
    return a if a > b else

# Taking input
a = input('Enter a number')
b = input('Enter another number')

# Passing the arguements to the function
greatest = greater(a, b)
print(greatest)
```

It's not that big of a problem for a single function, but assume that you're creating a python module that accepts certain data types, it'd be tiresome for users to convert the data type every time before passing to your module or you'd have to repeat your code just for converting the data types.


# What are Type Hints?

Type hints in python is a way of telling python the type of data you'd want to accept and return. It is mostly used in functions, but you can use it while declaring variables as well. 

## Why use Type Hints?

- Using type hints will provide better autocompletion in your favorite IDE.
- You won't have to check the documentation for knowing the return type and the data type of parameters in a function.
- Type Hints make it easier to write and generate documentation.

Python being a dynamically typed language, type hints don't interfere with the core principles of python. Type hints won't affect the running of the code during runtime. Type hinting as `str` and passing `int` is perfectly okay. 

> **NOTE:** Type hint is not something that executes. Python ignores your type hints while running your code. 

## How to use Type Hints?

### Code without Type Hints
```py
def func_a(a, b):
    return a + b
```

### Code with Type Hints
```py
def func_a(a: int, b: int) -> int:
    return a + b
```

Explanation:
- `a: int` means that the parameter a needs to be a `int` type.
- `-> int` means that the function `func_a` returns a value with type `int`


## The `typing` python module

This module provides runtime support for type hints. The most fundamental support consists of the types Any, Union, Callable, TypeVar, and Generic. 

### What is `typing.Any`?

`Any` is a special type indicating an unconstrained type. Every type is compatible with `Any`. `Any` is compatible with every type. It is used when you don't know the exact type of data, and the type of data may be anything.

### What is `typing.Callable`?

`Callable` is used to denote something that can be called. Most often, it is used to denote a function. `Callable[[int, int], str]` means that the provided function will take 2 integer arguments and return a value with the type of `str`. Checkout [PEP 677](https://www.python.org/dev/peps/pep-0677/) for more info.

### What is `typing.TypeVar`?

`TypeVar` is used for creating dynamic type hints. It is mainly used by static type checkers and IDE for proper autocompletion. Check out [Python Docs](https://docs.python.org/3/library/typing.html#typing.TypeVar) for more info.

> For a detailed introduction to type hints, see [PEP 483](https://www.python.org/dev/peps/pep-0483/).

# Converting the type on RunTime

## Creating a function for conversion

This is a good solution, but it's only viable if there are only a few parameters in a function.

Example:

```py
import typing as t

# Creating a TypeVar _T 
_T = TypeVar('_T')

# Creating a function that converts the type 
def convert_type(arg: t.Any, converter: _T) -> _T:
    return converter(arg)
    
# Function that sqaures the given number
def square(number: t.Any) -> int:
    # Converting str -> int
    number = convert_type(number, int)
    return number ** 2
```

Code Explanation:

- Created a [TypeVar](https://docs.python.org/3/library/typing.html#typing.TypeVar) (`_T`). You can use `TypeVar` for dynamic type hints. `convert_type` function takes the `converter` with type `_T` and the returns value of type `_T`. So if we pass `str` in `converter`, the function would show `str` as the return value. 
- Created the function `convert_type` which accepts `arg` and `converter` as it's 2 parameters.
- Created a function square that squares the number. But as the passed value is `str`, `convert_type(number, int)` is used to convert the number from `str` to `int`.

The code seems good to use everywhere, but once we increase the number of parameters in the function, it starts to have the same problems like **repetition of code** and **less readability** as [mentioned above](#heading-introduction).


## Creating a decorator for conversion

### What is a decorator?

Python has an interesting feature called [decorators](https://www.geeksforgeeks.org/decorators-in-python/) to add functionality to an existing code. This is also called [metaprogramming](https://stackoverflow.com/questions/514644/what-exactly-is-metaprogramming) because a part of the program tries to modify another part of the program at `runtime`.

Simply put, decorators are just a sugar-coated version of using one function on top of another function.

#### Code Without Decorator

```py
import typing as t

# Creating a function which takes another function
# as the parameter
def make_pretty(func: t.Callable) -> t.Callable:
    # Running the passed function after printing
    print("I got prettier")
    func()

# Defining the actual function
def print_something() -> None:
    print('something')

# Passing the actual function inside make_pretty()
make_pretty(print_something)
```

#### Code With Decorator

```py
import typing as t

# Creating the decorator make_pretty 
# It takes the function automatically as the
# paramater if you use the @decorator syntax
def make_pretty(func: t.Callable) -> t.Callable:
    # This inner function takes all the arguments (args)
    # and keyword arguments (kwargs) passed to the 
    # function we decorate. It can be None as well
    def inner_function(*args, **kwargs):
        # We can do anything we want here
        print("I got prettier")
        # Finally running the passed function
        func(*args, **kwargs)
    # Returning the inner function so that
    # decorator can use it properly
    return inner_function


# using @make_pretty decorates print_something()
# function with make_pretty() function
@make_pretty
def print_something() -> None:
    print("something")

# Actually running the function
print_something()
```

Explanation:

- Usage of `function(*args)` means if we call `function(1, 2, 3, 4, 5)`, the value of the arguement `args` would be `(1, 2, 3, 4, 5)` **{Tuple}** and vice-versa.
- Usage of `function(**kwargs)` means if we call `function(a=1, b=2, c=3)`, the value of the arguement `kwargs` would be `{a: 1, b: 2, c: 3}` **{Dictionary}** and vice-versa.
- If nothing is passed, the values of `args` and `kwargs` will remain `None`.
- `@` is used to denote the usage of a `decorator`.

### Getting type hints of a function

For converting the `argument` type according to the type hints in runtime, we'll have to get the type hints of the function.

```py
from typing import get_type_hints

def add_1(a: int) -> int:
    return a + 1

print(get_type_hints(add_1))
```

OUTPUT:

```
{'a': <class 'int'>, 'return': <class 'int'>}
```

### Creating a decorator for type conversion

```py
import typing as t

# Creating a TypeVar for dynamic type hinting in the function
_T = t.TypeVar("_T")

# Creating a function for converting type_conversion
def type_conversion(value: t.Any, value_type: type[_T]) -> _T:
    # If the value's type has a function named 'convert_types'
    # than we call the function and pass the value to it
    if hasattr(value_type, "convert_types"):
        return value_type.convert_types(value)
    # Returning the value if it's of Any type
    elif value_type is t.Any:
        return value
    # if it doesn't we return the value by forcefully
    # trying to convert
    return value_type(value)


# Creating a decorator to convert types
def convert_types(func: t.Callable) -> t.Callable:
    def inner_function(*args, **kwargs) -> t.Any:
        # Getting the type hints of the function
        annotations = t.get_type_hints(func).copy()
        # Removing the 'return' from the dictioary
        # as it is unnecessary right now
        annotations.pop("return")

        # Getting only the values of annotations for
        # looping with the args of the function
        args_annotations = annotations.values()
        new_args = list(args)

        # Looping through the annotations and args
        for index, (arg, arg_type) in enumerate(zip(args, args_annotations)):
            # If arg is not of the required type, we convert it
            if not isinstance(arg, arg_type):
                new_args[index] = type_conversion(arg)

        for arg_value, (arg_name, arg_type) in zip(kwargs, annotations.items()):
            # If arg is not of the required type, we convert it
            if not isinstance(arg_value, arg_type):
                kwargs[arg_name] = type_conversion(arg_value)

        return func(*new_args, **kwargs)

    return inner_function


# Creating a simple function to add 1
@convert_types
def add_1(a: int) -> int:
    return a + 1


# Passing a string to the function
print(add_1("1234"))
```

OUTPUT: 
```
1235
```


#### Explanation of type checker code

- We created a function `type_conversion` for converting the type of the value according to the passed required type. This function first checks if the type has a method `convert_types` and if it does, the value is passed onto that method. Else if the value is of type `typing.Any`, only the value is returned else the value is forcefully tried to convert.
- The `convert_types` decorator gets the type hints of the function, checks if the type hints are matching with the value types, and it's not, the values' type is changed. 
- `zip` function is used to combine 2 iterables. Zipping 2 tuples; `(1, 2, 3)` and `('a', 'b', 'c')` will produce a zip object with value `((1, 'a'), (2, 'b'), (3, 'c'))`.
- `enumerate` function is used to get the value and index while looping through the iterable at once. 
- `isinstance` is used to check if the value is an instance of the passed class.

### Making CustomClass compatible with type checker

While it may seem like the above type checker only supports the built-in data types in python. It's not actually true. We can make our own classes compatible with the type checker by simply adding a `convert_types` `@classmethod`.

```py
import typing as t

class MyOwnClass:
    @classmethod
    def convert_types(cls, value: t.Any) -> 'MyOwnClass':
        return str(value)
```
> **NOTE:** Yes, you can type-hint as strings as well instead of actual objects. You can use `-> 'int'` instead of `-> int` as per your use case. 

Now you can type-hint your parameters with `MyOwnClass` and it'd still work. 

## When not to use this decorator?

- If you have something complex like `t.Union` types; it'd be better to use [pydantic](https://pydantic-docs.helpmanual.io/).
- If you're writing something that is time-sensitive, it'd be better to stay away from runtime type checking, or this specific implementation of runtime type checking, as it loops 2 times before even executing your function.

# Resources for further reading

- [Python Typing Module](https://docs.python.org/3/library/typing.html)
- [Complete guide to python type checking](https://realpython.com/python-type-checking/)
- [Detailed introduction to type hints](https://www.python.org/dev/peps/pep-0483/)

# Conclusion

Python type hints were a game changer to developer experience ever since it was introduced in `python 3.5`. Python runtime checking is a fun topic to experiment with and might have a very wide range of use cases. But one must use it with extreme caution as it may introduce unexpected slowdowns and bugs to the code. 

# Connect With Me
- [Github](https://github.com/CaffeineDuck)
- [Twitter](https://twitter.com/Caffe1neDuck)
- [Linkedin](https://www.linkedin.com/in/caffeineduck/)
- [Facebook](https://www.facebook.com/Samrid.pandit12/)

