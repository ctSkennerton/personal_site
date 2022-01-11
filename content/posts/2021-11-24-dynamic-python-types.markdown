---
title: 'Dynamic types in Python.'
date: Wed, 24 Nov 2021 01:45:00 -0800
draft: false
tags: ['python']
---

I'm constantly learning new things about the Python language. I
consider myself a pretty good python programmer but often you never
need to use all of the language features when writing your own code.
For example I've not used is the class factory pattern using the
`type` built in function. I've been aware of class factories, and
read a few blog posts but never *grokked* it, until now.

A class factory is a function or another class that can create
classes at runtime rather than you writing out the class definition
in code. In other words it's a way to programatically create classes.
Normally when you write python you create classes in your code like
so:

```python
class A:
    def __init__(self):
        self.x = 1
        self.y = 2


a = A()
print(a.x, a.y)
```

So above, we define the name of a class, `A`, and give it an
initialization method. Then we create and instance of that class,
`a`, and print its two instance variables. There is another way to
create this class though using the [`type` function](https://docs.python.org/3/library/functions.html#type),
which lets you create types (classes). We could reconstruct the
code above as follows:

```python
def init_func(self):
    self.x = 1
    self.y = 2
    
A = type('A', (), {"__init__": init_func})

a = A()
print(a.x, a.y)
```

The fun bit is the third argument to `type` which defines a dictionary
where the keys are the names of methods (and class variables) for
the class. The values of the dictionary are either function names
or values. Note that for `init_func` you don't want to add the
parentheses as this will call the function (and give an error). For
me, this also helped illuminate why in python we have to pass `self`
as the first argument to a method -- because you can attach on any
old function to a class. It also goes to show that `self` is just
a convention. If you were so inclined you could make python look
like another language that uses `this` and not `self`:

```python
# Let's make python look a little more like C++
# which uses "this" instead of "self"
# Probably don't want to do this for real as it
# will really confuse people
def constructor(this):
    this.x = 1
    this.y = 2

B = type('Cpp', (), {'__init__': constructor})
b = B()
print(b.x, b.y)
```

So now lets use this knowledge to make our "factory". Take most of
the code above and wrap it in another function. This way you can
pass configuration for your to-be-created class into the outer
function. You can extend this as far as you want and don't have to
make it a function either, you could create a class whose job it
is to create other classes -- a class factory class!

```python
def class_factory(name='A'):
    def init_func(self):
        self.x = 1
        self.y = 2
        
    return type(name, (), {"__init__": init_func})

A = class_factory()
a = A()
print(a.x, a.y)
```

This is all nice and abstract but how and why would you use this?

Lets have a look at the
[marshmallow](https://marshmallow.readthedocs.io/en/stable/) library
which performs object serialization. With marshmallow you can define
incoming data, usually in the form of JSON and convert it to native
python data types (and vice-versa) while validating the data conforms
to a certain schema. Here is the example from their website:

```python
from datetime import date
from pprint import pprint

from marshmallow import Schema, fields


class ArtistSchema(Schema):
    name = fields.Str()


class AlbumSchema(Schema):
    title = fields.Str()
    release_date = fields.Date()
    artist = fields.Nested(ArtistSchema())


bowie = dict(name="David Bowie")
album = dict(artist=bowie, title="Hunky Dory", release_date=date(1971, 12, 17))

schema = AlbumSchema()
result = schema.dump(album)
pprint(result, indent=2)
# { 'artist': {'name': 'David Bowie'},
#   'release_date': '1971-12-17',
#   'title': 'Hunky Dory'}
```

The key things to note are that you have to create schema classes
that define fields that are of particular types. But what if you
don't know the types beforehand or the types changed outside of the
control of your program? For example, you have a web service that
accepts incoming data, but the input data is user defined (to a
point) and the actual schema of the data are stored outside of your
program. It's not feasible for you to code in a new class in your
web service every time the data changes but you also don't want to
accept any old user data. What you really need is to read in one
of these external schemas to create a validator for the input data
on the fly.

Class factories to the rescue! Lets see how we can build a marshmallow
schema class from a simple definition stored in JSON as a dictionary
of field names to types.


```python
from typing import Dict
from marshmallow import Schema, fields

def schema_factory(schema_definition: Dict[str, str]):
    """
        The schema definition is a simple dictionary with the name
        of the field as the key and a string that defines its type
        as the value. For example:
        
        {
            'field1': 'str',
            'field2': 'date'
        }
    """
    fields = {}
    field_type_map = {'str': fields.Str,
        'date': fields.Date
        # add in other types as needed
    }
    for schema_field_name, schema_field_value in schema_definition.items():
        # We find the right marshmallow field and initialize it.
        # Fields then defines the class variables that you would 
        # manually do when defining a marshmallow Schema.
        fields[schema_field_name] = field_type_map[schema_field_value]()
        
    return type('CustomSchema', (Schema,), fields)

```
