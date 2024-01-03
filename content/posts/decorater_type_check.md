---
title: Implementing Type Checker in Python3
date: 2018-03-20
tags:
- Python
- Static Type
- Type Hint
categories:
- implement
description: "Type hint and type checker"
---

## introduction
As we all know, python is a dynamic language and dynamic typing can be flexible, powerful, convenient and easy. But with your project growing, dynamic typing is not always the best approach. As an opposite, static typing can make programs easier to understand and maintain.
- Type declarations can serve as machine-checked documentation. This is important as code is typically read much more often than modified, and this is especially important for large and complex programs.
- Static typing can help you find bugs earlier and with less testing and debugging. Especially in large and complex projects this can be a major time-saver.
- Static typing can help you find difficult-to-find bugs before your code goes into production. This can improve reliability and reduce the number of security issues.
- Static typing makes it practical to build very useful development tools that can improve programming productivity or software quality, including IDEs with precise and reliable code completion, static analysis tools, etc.
- You can get the benefits of both dynamic and static typing in a single language. Dynamic typing can be perfect for a small project or for writing the UI of your program, for example. As your program grows, you can adapt tricky application logic to static typing to help maintenance.

[PEP 3107](https://www.python.org/dev/peps/pep-3107/) introduces a syntax for adding arbitrary metadata annotations to Python functions. Annotations for parameters take the form of optional expressions that follow the parameter name:
```
def foo(a: expression, b: expression = 5):
```
 [PEP484](https://www.python.org/dev/peps/pep-0484/#abstract) introduces a provisional module to provide these standard definitions and tools, along with some conventions for situations where annotations are not available.
 ```
 def greeting(name: str) -> str:
    return 'Hello ' + name

 ```
 While these annotations are available at runtime through the usual __annotations__ attribute, no automatic type checking happens at runtime. Instead, it is assumed that a separate off-line type checker (e.g. mypy) will be used for on-demand source code analysis.

Until Now, there is no specific methods to do the type check only some tools such as `mypy`, `PyCharm`. So here, we use create a simple type checker.

## implement
Firstly, we have a simple function. This is the normal one.
```
def gcd(a, b):
    '''Return the greatest common divisor of a and b.'''
    a = abs(a)
    b = abs(b)
    if a < b:
        a, b = b, a
    while b != 0:
        a, b = b, a % b
    return a

```
In this example, we consider a and b are both `int` type. Then, we add the annotations to the function `gcd`.
```
def gcd(a: int, b: int) -> int:

```
we can use `gcd.__annotations__` to get the information.
```
gcd.__annotations__
{'return': <class 'int'>, 'b': <class 'int'>, 'a': <class 'int'>}

```

then we create a decorater -- `_type_check.py`  , the basic idea is that using `isinstance` method to test all the types in the annotations with the real types. If it doesn't match, alert some error message.
```
import functools
def typecheck(f):
    @functools.wraps(f)
    def wrapper(*args, **kwargs):
        print("hello the _type_check")
        for i, arg in enumerate(args[:f.__code__.co_nlocals]):
            name = f.__code__.co_varnames[i]
            expected_type = f.__annotations__.get(name, None)
            if expected_type and not isinstance(arg, expected_type):
                raise RuntimeError("error: Argument {} to '{}' has incompatible type '{}' ; expected '{}'".format( i+1 , f.__name__, type(arg).__name__, expected_type.__name__))
                # raise RuntimeError("{} should be of type {}; {} specified".format(name, expected_type.__name__, type(arg).__name__))
        for name, arg in kwargs.items():
            expected_type = f.__annotations__.get(name, None)
            if expected_type and not isinstance(arg, expected_type):
                raise RuntimeError("error: Argument {} to '{}' has incompatible type '{}' ; expected '{}'".format( i+1 , f.__name__, type(arg).__name__, expected_type.__name__))
        result = f(*args, **kwargs)
        return_type = f.__annotations__.get('return', None)
        if return_type and not isinstance(result, return_type):
            raise RuntimeError("{} should return {}".format(f.__name__, return_type.__name__))
        return result
    return wrapper


```
Now, the gcd.py is going to be like this:
```
from _type_check import typecheck

@typecheck
def gcd(a: float, b: float) -> int:
    '''Return the greatest common divisor of a and b.'''
    a = abs(a)
    b = abs(b)
    if a < b:
        a, b = b, a
    while b != 0:
        a, b = b, a % b
    return a


```
In the end, we create some test:
```
from gcd import gcd
gcd(2,3)
gcd(2.2,'str')

```
we can get the result below, which report some details about the wrong type.
```
RuntimeError: error: Argument 1 to 'gcd' has incompatible type 'int' ; expected 'float'
RuntimeError: error: Argument 2 to 'gcd' has incompatible type 'str' ; expected 'float'

```

## conclusion
This method just a simple type check, which can do the basic things -- comparing the annotation types with the real types and reporting the wrong messages. It is obviously that it has a lot of things to do compared with the famous type checker tool `mypy`, Such as:
- how to deal with the extra parameters
- how to support several types
- Here we use the raise RuntimeError to report the error, it means when it met the error, it will stop. Is there some better method to log the messages.
