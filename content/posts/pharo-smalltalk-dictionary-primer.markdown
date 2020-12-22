---
title: 'Using Dictionaries in Pharo Smalltalk'
date: Mon, 21 Dec 2020 13:54:32 -0800
tags: ['Pharo', 'Smalltalk']
---

Starting out with Smalltalk can be a little jarring as it doesn't have the similar syntax as launguages that
are more heavily inspired by C. Dictionaries are one kind of data structure where I noticed this the most so
I put together my notes on using them in Pharo with some comparisons to Python. In many other languages there
 is a *subscript operator* that allows you to access a value in a dictionary
(and also a position in an array). It's often written `dict[key]` to access a value and `dict[key]=val` to 
set a value to an existing key or add in a new key. Smalltalk doesn't contain
a subscript operator and so interacting with dictionaries requires a slight change 
in the mental model to get things done. 

Dictionaries in Pharo are composed of Associations. An Association is a class that holds a single key&ndash;value
pair. You can create an Association by sending the `->` binary message to an object (e.g. `1 -> 2`). Associations
can be used outside of dictionaries as well, for example the following code snippet creates an Array of Associations:

```smalltalk
{ 1 -> 2. 3 -> 4. 5 -> 6 }
```

## Initialization

You can create a Dictionary from an Array of Associations using the `newFrom:` message

```smalltalk
Dictionary newFrom: { 1 -> 2. 3 -> 4. 5 -> 6 }.
```
or you can achieve the same thing with the `at:put:` message:
```smalltalk
Dictionary new 
	at: 1 put: 2; 
	at: 3 put: 4; 
	at: 5 put: 6; 
	yourself.
```
While the `newFrom:` message seems intuative to me, the `at:put:` way of creating Dictionaries took me
a while to get my head around. First,
notice the semicolons between the successive calls to `at:put:`, this is for cascading messages that should
all go to the same "receiver", which in this case is the Dictionary object created by `new`. What happens
if you don't put in the semicolons? Pharo will get confused and think you are trying to send a single
message called `at:put:at:put:at:put:` instead of three separate messages and you'll get an error.
 Why is there no semicolon after
`new`? So what's happening is that the `new` message is creating an instance of the Dictionary class i.e.
a Dictionary object which gets returned from the `new` message. The `at:put:` messages are then applied to
the return value of `new`. If you put a semicolon after `new` then all of the subsequent messages will be 
sent to what `new` was sent to, rather than the result of `new`. In other words, `at:put:` will be sent to
the Dictionary class and not a Dictionary object (which causes an error). The last bit of the statement, `yourself`,
is needed to return the Dictionary object. Without `yourself` the return value of the final `at:put:` message
is used, which is the value added to the dictionary. If the return value of `at:put:` was the dictionary
object then the `yourself` message wouldn't be needed at all.

## Accessing elements

The basic method of getting values is to use the `at:` message:
```smalltalk
d := Dictionary newFrom: { 1 -> 2. 3 -> 4. 5 -> 6 }.

d at: 3.  ">>> 4"
```
Just like in Python, if you try to access a key that doesn't exist you'll get an error
```smalltalk
d at: 100.   "Error KeyNotFound"
```
but unlike Python there are multiple ways to avoid this error using variants of the `at:` message.
The `at:update:initial:` message allows you either update or set a value in a dictionary:
```smalltalk
d at: 1 update: [ :value | value + 1 ] initial: [ 1 ].
```
which is broadly equivelent to the following Python code
```python
try:
    d[1] += 1
except KeyError:
    d[1] = 1
```

There are many other variants of `at:` that modify the behaviour depending on whether the key is present
or not.

## Enumerating

Enumerating in Pharo Smalltalk can also be achieved in multiple ways. The `do:` message is available in many
classes for iterating, which for Dictionaries iterates through the values:
```smalltalk
d do: [ :value | Transcript show: value; cr]
```
which is Python would be:
```python
for value in d.values():
    print(value)
```
The `do:` message is an alias for `valuesDo:` and there is also a `keysDo:` for iterating through the keys
and `associationsDo:` for iterating through the key&ndash;value pairs. Unlike Python which returns a tuple
of the key and value, in Pharo an Association object is returned. This class responds to the `key` and `value`
messages for accessing each.
```smalltalk
d associationsDo: [ :pair | Transcript show: pair key; cr; show: pair value; cr]
```
and the Python equivelent:
```python
for pair in d.items():
    print(pair[0])
    print(pair[1])
```

Despite the differences introduced through the syntax of each language, using dictionaries in Pharo Smalltalk
and Python are basically the same.
