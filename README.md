# Magic Method in Python

## Arithmetic Operators

Operator - Supporting Method

- `+`       `.__add__(self, other)`
- `-`	    `.__sub__(self, other)`
- `*`	    `.__mul__(self, other)`
- `/`	    `.__truediv__(self, other)`
- `//`	    `.__floordiv__(self, other)`
- `%`	    `.__mod__(self, other)`
- `**`	    `.__pow__(self, other[, modulo])`

Here’s a summary of the .__r*__() methods:
Python calls these methods when the left-hand operand doesn’t support the corresponding operation and the operands are of different types.

- `+`       `.__radd__(self, other)`
- `-`	    `.__rsub__(self, other)`
- `*`	    `.__rmul__(self, other)`
- `/`	    `.__rtruediv__(self, other)`
- `//`	    `.__rfloordiv__(self, other)`
- `%`	    `.__rmod__(self, other)`
- `**`	    `.__rpow__(self, other[, modulo])`

Python also has some unary operators:

- `-`	`.__neg__(self)`	Returns the target value with the opposite sign
- `+`	`.__pos__(self)`	Provides a complement to the negation without performing any transformation


## Comparison Operator Methods

- `<`	`.__lt__(self, other)`
- `<=`	`.__le__(self, other)`
- `==`	`.__eq__(self, other)`
- `!=`	`.__ne__(self, other)`
- `>=`	`.__ge__(self, other)`
- `>`	`.__gt__(self, other)`

```python
class Rectangle:
    def __init__(self, height, width):
        self.height = height
        self.width = width

    def area(self):
        return self.height * self.width

    def __eq__(self, other):
        return self.area() == other.area()

    def __lt__(self, other):
        return self.area() < other.area()

    def __gt__(self, other):
        return self.area() > other.area()
```

### Membership Operators

```python
class Stack:
    def __init__(self, items=None):
        self.items = list(items) if items is not None else []

    def push(self, item):
        self.items.append(item)

    def pop(self):
        return self.items.pop()

    def __contains__(self, item):
        for current_item in self.items:
            if item == current_item:
                return True
        return False
```

## Augmented Assignments

- `+=`	`.__iadd__(self, other)`
- `-=`	`.__isub__(self, other)`
- `*=`	`.__imul__(self, other)`
- `/=`	`.__itruediv__(self, other)`
- `//=`	`.__ifloordiv__(self, other)`
- `%=`	`.__imod__(self, other)`
- `**=`	`.__ipow__(self, other[, modulo])`

## Controlling Attribute Access

- `.__getattribute__(self, name)`	Runs when you access an attribute called name
- `.__getattr__(self, name)`	Runs when you access an attribute that doesn’t exist in the current object
- `.__setattr__(self, name, value)`	Runs when you assign value to the attribute called name
- `.__delattr__(self, name)`	Runs when you delete the attribute called name

### Deleting Attributes

```python
class NonDeletable:
    def __init__(self, value):
        self.value = value

    def __delattr__(self, name):
        raise AttributeError(
            f"{type(self).__name__} doesn't support attribute deletion"
        )
```

## Managing Attributes Through Descriptors


- `.__get__(self, instance, type=None)`	Getter method that allows you to retrieve the current value of the managed attribute
- `.__set__(self, instance, value)` Setter method that allows you to set a new value to the managed attribute
- `.__delete__(self, instance)`	Deleter method that allows you to remove the managed attribute from the containing class
- `.__set_name__(self, owner, name)`	Name setter method that allows you to define a name for the managed attribute

```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        self._name = name

    def __get__(self, instance, owner):
        return instance.__dict__[self._name]

    def __set__(self, instance, value):
        if not isinstance(value, int | float) or value <= 0:
            raise ValueError("positive number expected")
        instance.__dict__[self._name] = value
```
```python
import math

# ...

class Circle:
    radius = PositiveNumber()

    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return round(math.pi * self.radius**2, 2)

class Square:
    side = PositiveNumber()

    def __init__(self, side):
        self.side = side

    def area(self):
        return round(self.side**2, 2)

class Rectangle:
    height = PositiveNumber()
    width = PositiveNumber()

    def __init__(self, height, width):
        self.height = height
        self.width = width

    def area(self):
        return self.height * self.width
```
```python
>>> from shapes import Circle, Square, Rectangle

>>> circle = Circle(-10)
Traceback (most recent call last):
    ...
ValueError: positive number expected

>>> square = Square(-20)
Traceback (most recent call last):
    ...
ValueError: positive number expected

>>> rectangle = Rectangle(-20, 30)
Traceback (most recent call last):
    ...
ValueError: positive number expected
```

## Supporting Iteration With Iterators and Iterables

### Creating Iterators

The table below shows the methods that make up an iterator. They’re known as the iterator protocol:

- `.__iter__()`	Called to initialize the iterator. It must return an iterator object.
- `.__next__()`	Called to iterate over the iterator. It must return the next value in the data stream.

As you can see, both methods have their own responsibility. 
In `.__iter__()`, you typically return self, the current object. 
In `.__next__()`, you need to return the next value from the data stream in a sequence. 
This method must raise a `StopIteration` when the stream of data is exhausted. 
This way, Python knows that the iteration has reached its end.


```python
class FibonacciIterator:
    def __init__(self, stop=10):
        self._stop = stop
        self._index = 0
        self._current = 0
        self._next = 1

    def __iter__(self):
        return self

    def __next__(self):
        if self._index < self._stop:
            self._index += 1
            fib_number = self._current
            self._current, self._next = (
                self._next,
                self._current + self._next,
            )
            return fib_number
        else:
            raise StopIteration
```

### Building Iterables

Iterables are a bit different from iterators. 
Typically, a collection or container is iterable when it implements the `.__iter__()` special method.

For example, Python built-in container types—such as lists, tuples, dictionaries, and sets—are iterable objects. 
They provide a stream of data that you can iterate over. 
However, they don’t provide the `.__next__()` method, which drives the iteration process. 
So, to iterate over an iterable using a for loop, Python implicitly creates an iterator.

```python
class Stack:
    # ...

    def __iter__(self):
        return iter(self.items[::-1])
```

## Making Your Objects Callable

```python
class Factorial:
    def __init__(self):
        self._cache = {0: 1, 1: 1}

    def __call__(self, number):
        if number not in self._cache:
            self._cache[number] = number * self(number - 1)
        return self._cache[number]
```
```shell
>>> from factorial import Factorial

>>> factorial_of = Factorial()

>>> factorial_of(4)
24
>>> factorial_of(5)
120
>>> factorial_of(6)
720
```

## Implementing Custom Sequences and Mappings

- `.__getitem__()`	Called when you access an item using indexing like in `sequence[index]`
- `.__len__()`	Called when you invoke the built-in `len()` function to get the number of items in the underlying sequence
- `.__contains__()`	Called when you use the sequence in a membership test with the in or not in operator
- `.__reversed__()`	Called when you use the built-in `reversed()` function to get a reversed version of the underlying sequence

```python
class Stack:
    # ...

    def __contains__(self, item):
        for current_item in self.items:
            if item == current_item:
                return True
        return False

    # ...

    def __getitem__(self, index):
        return self.items[index]

    def __len__(self):
        return len(self.items)

    def __reversed__(self):
        return type(self)(reversed(self.items))
```
```shell
>>> from stack import Stack

>>> stack = Stack([1, 2, 3, 4])

>>> stack[1]
2
>>> stack[-1]
4

>>> len(stack)
4

>>> reversed(stack)
Stack([4, 3, 2, 1])
```


## Handling Setup and Teardown With Context Managers

- `.__enter__()`	Sets up the runtime context, acquires resources, and may return an object that you can bind to a variable with the as specifier on the with header
- `.__exit__()`	Cleans up the runtime context, releases resources, handles exceptions, and returns a Boolean value indicating whether to propagate any exceptions that may occur in the context

```python
class TextFileReader:
    def __init__(self, file_path, encoding="utf-8"):
        self.file_path = file_path
        self.encoding = encoding

    def __enter__(self):
        self.file_obj = open(self.file_path, mode="r", encoding=self.encoding)
        return self.file_obj

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.file_obj.close()
```




## Credits

- [Python's Magic Methods: Leverage Their Power in Your Classes](https://realpython.com/python-magic-methods)
