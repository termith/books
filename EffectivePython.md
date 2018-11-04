# Effective Python

## Python thinking

### Strings and Unicode

* In Python 3, bytes contains sequences of 8-bit values, str contains sequences of Unicode characters. bytes and str instances can’t be used together with operators (like > or +)
* In Python 2, str contains sequences of 8-bit values, unicode contains sequences of Unicode characters. str and unicode can be used together with operators if the str only contains 7-bit ASCII characters
* Use helper functions to ensure that the inputs you operate on are the type of character sequence you expect (8-bit values, UTF-8 encoded characters, Unicode characters, etc.)
* If you want to read or write binary data to/from a file, always open the file using a binary mode (like 'rb' or 'wb').”
Representation
Function ```repr()``` returns a string containing a printable representation of an object. Class can control what this function returns using ```__repr__()``` method

### Slices and strides

* Specifying start, end, and stride in a slice can be extremely confusing
* Prefer using positive stride values in slices without start or end indexes. Avoid negative stride values if possible
* Avoid using start, end, and stride together in a single slice. If you need all three parameters, consider doing two assignments (one to slice, another to stride) or using ```islice``` from the ```itertools``` built-in module

### List comprehensions and generators

```even_squares = [x**2 for x in a if x % 2 == 0]```

* List comprehensions are clearer than the ```map``` and ```filter``` built-in functions because they don’t require extra lambda expressions
* List comprehensions allow you to easily skip items from the input list, a behavior ```map``` doesn’t support without help from filter
* Dictionaries and sets also support comprehension expressions

```python
chile_ranks = {'ghost': 1, 'habanero': 2, 'cayenne': 3}
rank_dict = {rank: name for name, rank in chile_ranks.items()}
chile_len_set = {len(name) for name in rank_dict.values()}

matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [x for row in matrix for x in row]
```

* A generator expression is created by putting list-comprehension-like syntax between () characters
 ```it = (len(x) for x in open('/tmp/my_file.txt'))```
* Generator expressions can be composed by passing the iterator from one generator expression into the for subexpression of another.
* Generator expressions execute very quickly when chained together

List iterators
* Prefer enumerate instead of looping over a range and indexing into a sequence
* You can supply a second parameter to ```enumerate``` to specify the number from which to begin counting (zero is the default).

* The ```zip``` built-in function can be used to iterate over multiple iterators in parallel
* “In Python 3, ```zip``` is a lazy generator that produces tuples. In Python 2, ```zip``` returns the full result as a list of tuples.
* zip truncates its output silently if you supply it with iterators of different lengths.
* The ```zip_longest``` function from the ```itertools``` built-in module lets you iterate over multiple iterators in parallel regardless of their lengths”

### Generators vs Lists

* Using generators can be clearer than the alternative of returning lists of accumulated results
* The iterator returned by a generator produces the set of values passed to yield expressions within the generator function’s body 

* Beware of functions that iterate over input arguments multiple times. If these arguments are iterators, you may see strange behavior and missing values.
* Python’s iterator protocol defines how containers and iterators interact with the ```iter``` and ```next``` built-in functions, for loops, and related expressions.
* You can easily define your own iterable container type by implementing the ```__iter__``` method as a generator.
* You can detect that a value is an iterator (instead of a container) if calling ```iter``` on it twice produces the same result, which can then be progressed with the next built-in function

### Else after loops

* The ```else``` block after a loop only runs if the loop body did not encounter a ```break``` statement.
* Avoid using ```else``` blocks after loops because their behavior isn’t intuitive and can be confusing.

### Try/Except/Else/Finally

* The ```else``` block helps you minimize the amount of code in try blocks and visually distinguish the success case from the ```try/except``` blocks.
* Use ```try/except/else``` to make it clear which exceptions will be handled by your code and which exceptions will propagate up. When the ```try``` block doesn’t raise an exception, the else block will run. 
* An ```else``` block can be used to perform additional actions after a successful try block but before common cleanup in a ```finally``` block

## Functions

### Closures

```python
def sort_priority(values, group):
    def helper(x):
        if x in group:
            return (0, x)
        return (1, x)
    values.sort(key=helper)
```

__How it works:__
Python has specific rules for comparing tuples. It first compares items in index zero, then index one, then index two, and so on. This is why the return value from the helper closure causes the sort order to have two distinct groups

When you reference a variable in an expression, the Python interpreter will traverse the scope to resolve the reference in this order:
1. The current function’s scope
2. Any enclosing scopes (like other containing functions)
3. The scope of the module that contains the code (also called the global scope)
4. The built-in scope (that contains functions like len and str)

* By default, closures can’t affect enclosing scopes by assigning variables
* In Python 3, use the nonlocal statement to indicate when a closure can modify a variable in its enclosing scopes
* In Python 2, use a mutable value (like a single-item list) to work around the lack of the nonlocal statement
* Avoid using nonlocal statements for anything beyond simple functions

### Default arguments

* Default arguments are only evaluated once: during function definition at module load time. This can cause odd behaviors for dynamic values (like ```{}``` or ```[]```).
* Use ```None``` as the default value for keyword arguments that have a dynamic value. Document the actual default behavior in the function’s docstring

### Keyword-only arguments

* Keyword arguments make the intention of a function call more clear
* The ```*``` symbol in the argument list indicates the end of positional arguments and the beginning of keyword-only arguments.
* Use keyword-only arguments to force callers to supply keyword arguments for potentially confusing functions, especially those that accept multiple Boolean flags
* Python 3 supports explicit syntax for keyword-only arguments in functions.
* Python 2 can emulate keyword-only arguments for functions by using ```**kwargs``` and manually raising ```TypeError``` exceptions

### Iterators

An iterator only produces its results a single time. If you iterate over an iterator or generator that has already raised a ```StopIteration``` exception, you won’t get any results the second time around.


### Hooks

__Hooks for defaultdict examples:__
```python
def log_missing():
   print('Key added')
   return 0
   
current = {'green': 12, 'blue': 3}
increments = [('red', 5), ('blue', 17), ('orange', 9)]
 
result = defaultdict(log_missing, current)
print('Before:', dict(result))
for key, amount in increments:
    result[key] += amount
print('After: ', dict(result))
 
>>>
Before: {'green': 12, 'blue': 3}
Key added
Key added
After:  {'orange': 9, 'green': 12, 'blue': 20, 'red': 5}


def increment_with_report(current, increments):
    added_count = 0
 
    def missing():
      nonlocal added_count  # Stateful closure
      added_count += 1
      return 0
 
    result = defaultdict(missing, current)
    for key, amount in increments:
        result[key] += amount
 
return result, added_count

added_count = 2
```

__Using object as hook:__

```python
class BetterCountMissing(object):
    def __init__(self):
        self.added = 0
        
    def __call__(self):
        self.added += 1
return 0
 
counter = BetterCountMissing()
result = defaultdict(counter, current)  # Relies on __call__
for key, amount in increments:
    result[key] += amount
assert counter.added == 2
```

### Alternative constructor

```@classmethod``` decorator provides alternative constructors for class.
See this question for more info.

### Mix-in

* Avoid using multiple inheritance if mix-in classes can achieve the same outcome.
* Use pluggable behaviors at the instance level to provide per-class customization when mix-in classes may require it.
* Compose mix-ins to create complex functionality from simple behaviors

### Private attributes

* ```__private_var``` of ```MyClass``` isn’t available from ```MyChildClass(MyClass)``` instances. 
But you can get access from it as ```_MyClass__private_var```.
See also ```MyClass.__dict__```

### ```@property```

* Use ```@property``` to give existing instance attributes new functionality

### Descriptors and lazy attributes
* ```__get__``` and ```__set__``` descriptors called when you want to get access to class attribute. So you have to in advance define attributes you want to access
* Use ```__getattr__``` and ```__setattr__``` to lazily load and save attributes for an object.
* Understand that ```__getattr__``` only gets called once when accessing a missing attribute, whereas ```__getattribute__``` gets called every time an attribute is accessed.
* Avoid infinite recursion in ```__getattribute__``` and ```__setattr__ ```by using methods from ```super()``` (i.e., the ```object``` class) to access instance attributes directly

### ```super``` statement

In Python3 you can use ```super``` without class and self

```python
class Explicit(MyBaseClass):
    def __init__(self, value):
        super(__class__, self).__init__(value * 2)

class Implicit(MyBaseClass):
    def __init__(self, value):
        super().__init__(value * 2)
```

### private fields

```python
class MyParentObject(object):
    def __init__(self):
        self.__private_field = 71”

class MyChildObject(MyParentObject):
    def get_private_field(self):
        return self.__private_field

baz = MyChildObject()
baz.get_private_field()

>>>
AttributeError: 'MyChildObject' object has no attribute '_MyChildObject__private_field
```

The private attribute behavior is implemented with a simple transformation of the attribute name. When the Python compiler sees private attribute access 
in methods like MyChildObject.get_private_field, it translates ```__private_field``` to access ```_MyChildObject__private_field``` instead. In this example, ```__private_field``` was only defined in ```MyParentObject.__init__```, meaning the private attribute’s real name is ```_MyParentObject__private_field```. Accessing the parent’s private attribute from the child class fails simply because the transformed attribute name doesn’t match.
Knowing this scheme, you can easily access the private attributes of any class, from a subclass or externally, without asking for permission
```python
baz._MyParentObject__private_field == 71
```

### ```collections.abc```

* Inherit directly from Python’s container types (like list or dict) for simple use cases.
* Beware of the large number of methods required to implement custom container types correctly.
* Have your custom container types inherit from the interfaces defined in collections.abc to ensure that your classes match required interfaces and behaviors.”

### Descriptors

```python
class Grade(object):
    def __get__(*args, **kwargs):
        # ...

    def __set__(*args, **kwargs):
        # ...

class Exam(object):
    # Class attributes
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()
```

When you assign a property:

```python
exam = Exam()
exam.writing_grade = 40
```

it will be interpreted as:

```python
Exam.__dict__['writing_grade'].__set__(exam, 40)
```

When you retrieve a property:

```python
print(exam.writing_grade)
```

it will be interpreted as:

```python
print(Exam.__dict__['writing_grade'].__get__(exam, Exam))
```

**But this is wrong!**

The problem is that a single Grade instance is shared across all Exam instances for the class attribute writing_grade. The Grade instance for this attribute is constructed once in the program lifetime when the Exam class is first defined

### ```__gettattr__, __setattr__, __getattribute__```

```__getattr__``` runs once to do the hard work of loading a property; all subsequent accesses retrieve the existing result

```__getattribute__```. This special method is called every time an attribute is accessed on an object, even in cases where it does exist in the attribute dictionary

### Validating with metaclass

```python
class ValidatePolygon(type):
    def __new__(meta, name, bases, class_dict):
        # Don't validate the abstract Polygon class
        if bases != (object,):
            if class_dict['sides'] < 3:
                raise ValueError('Polygons need 3+ sides')
        return type.__new__(meta, name, bases, class_dict)

class Polygon(object, metaclass=ValidatePolygon):
    # for python2
    # __metaclass__ = ValidatePolygon
    sides = None  # Specified by subclasses

    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180

class Triangle(Polygon):
    sides = 3

```

If you try to define a polygon with fewer than three sides, the validation will cause the class statement to fail immediately after the class statement body. This means your program will not even be able to start running when you define such a class.
Click here to view code image

```python
print('Before class')
class Line(Polygon):
    print('Before sides')
    sides = 1
    print('After sides')
print('After class')

>>>
Before class
Before sides
After sides
Traceback ...
ValueError: Polygons need 3+ sides”
```

The ```__new__``` method of metaclasses is run after the ```class``` statement’s entire body has been processed

## Concurrency and Parallelism

* Python threads release the GIL just before they make system calls and reacquire the GIL as soon as the system calls are done.
* Even though Python has a global interpreter lock, you’re still responsible for protecting against data races between the threads in your programs.
* The ```threading.Lock``` class is Python’s standard mutual exclusion lock implementation.

### 3 problems with threads

1. They require special tools to coordinate with each other safely (e.g. ```Lock```, ```Queue```)
2. Threads require a lot of memory, about 8 MB per executing thread
3. Threads are costly to start

Python can work around all these issues with coroutines

```python
def my_coroutine():
    while True:
        received = yield
        print('Received:', received)

it = my_coroutine()
next(it)             # Prime the coroutine
it.send('First')
it.send('Second')

>>>
Received: First
Received: Second
```

The initial call to next is required to prepare the generator for receiving the first send by advancing it to the first yield expression

## Imports

When a module is imported, here’s what Python actually does in depth-first order:
1. Searches for your module in locations from sys.path
2. Loads the code from the module and ensures that it compiles
3. Creates a corresponding empty module object
4. Inserts the module into sys.modules
5. Runs the code in the module object to define its contents

## ```repr```

* Calling ```print``` on built-in Python types will produce the human-readable string version of a value, which hides type information.
* Calling ```repr``` on built-in Python types will produce the printable string version of a value. These repr strings could be passed to the ```eval``` built-in function to get back the original value.
* ```%s``` in format strings will produce human-readable strings like ```str```. ```%r``` will produce printable strings like ```repr```.
* You can define the ```__repr__``` method to customize the printable representation of a class and provide more detailed debugging information.
* You can reach into any object’s ```__dict__``` attribute to view its internals

## Profiling

* Use the ```cProfile``` module instead of the ```profile``` module because it provides more accurate profiling information
* The ```Profile``` object’s ```runcall``` method provides everything you need to profile a tree of function calls in isolation
* The ```Stats``` object lets you select and print the subset of profiling information you need to see to understand your program’s performance










