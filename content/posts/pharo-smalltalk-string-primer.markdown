---
title: 'Using Strings in Pharo Smalltalk'
date: Mon, 19 Jul 2021 01:20:17 -0700
tags: ['Pharo', 'Smalltalk']
---

Smalltalk syntax can be a little confusing coming from other languages. Here I'll show some comparisions 
between Python string operations and Smalltalk.

## Substrings / Slicing
Python strings use the slice notation where you can place up to three colon-separated values for the start, stop,
and step. Python strings are 0-indexed and the stop argument is one past the final element that you want.
```python
s = 'abcdefg'
s[1:]  # bcdefg
s[:2]  # ab
s[1:6]  # bcdef
s[1:-1]  # bcdef
```
The slice notation in Python is compact and versatile to be used when getting the beginning, middle, or end
of a string. Smalltalk achieves the same results using a number of different messages that can be a little 
confusing when you first get into it.

```smalltalk
s := 'abcdefg'

"Leave off only the first character"
s allButFirst  "bcdefg"

"allButFirst with an argument to leave off that many characters"
s allButFirst: 2  "cdefg"

"There are also the first and last messages that produce the same result
as s[1:] or s[:5]. Remember though that Smalltalk's indexing is 1-based 
and inclusive"
s first: 2  "ab"
s last: 2  "fg"

"Get the middle of the string"
s copyFrom: 2 to: 6  "bcdef"

"Smalltalk indexing doesn't have the same trick with negative indexes,
instead you need to calculate the size"
s copyFrom: 2 to: s size - 1 
```

## Formatting

Python has extensive string formatting using either the `.format()` method of string or using f-strings.
Pharo has a similar `format:` message that can be used for inpterpolating other objects in strings.
Each of these objects must have a `asString` method for this to work.

```smalltalk
"pass in a collection where values are indexed by number (like Array or OrderedCollection)"
'ab {1} ef {2}' format: {'cd'. 'gh'}.  "ab cd ef gh"

"pass in a collection where values are indexed by name"
'ab {one} ef {two}' format: 
    (Dictionary with: #one -> 'cd' with: #two -> 'gh').  "ab cd ef gh"
```

Unlike python there is no format mini-language. Which means that if you want to format things
like numbers in a specific way you need to do that before passing the object to `format:`.
Numbers have a basic message `printString` that will use the default settings. For very large or 
very small numbers it will print in scientific notation.

There isn't a general number formatter like `printf` in C/C++, instead there are messages for each
type of formatting you want on a number. `printPaddedWith: aCharacter to: aNumber` is used to
perform left-side padding with a character of your choice. Normally you would choose zero to
mimic the `printf` behaviour but you can choose any character you like.

```smalltalk
123 printPaddedWith: $0 to: 8     "'00000123'"
123 printPaddedWith: $, to: 8     "',,,,,123'"
123.31 printPaddedWith: $0 to: 6  "'000123.31'"
123.31 printPaddedWith: $0 to: 1  "'123.31'"
```

`printShowingDecimalPlaces: placesDesired` is for controlling the number of decimal places. 

```smalltalk
123.31 printShowingDecimalPlaces: 12  "'123.310000000000'"
```

To combine these with the `format:` message you could do something like the following to make
a dynamic array of the values you want:

```smalltalk
'{1}' format: {  1.123 printShowingDecimalPlaces: 12. }
```

There isn't a scientific formatter though, instead the default is to print a number in decimal
format when it's small, and switch to the scientific format as it gets bigger. I found this
a little annoying as I wanted to print numbers in a consistent scientific format no matter their
size. 

There is a package called [NumberPrinter](http://ss3.gemstone.com/ss/NumberPrinter/) that has more 
versatile printing options, including scientific printing. For example, the default settings with
`FloatPrinterScientificFormat` would convert 123.123 to 1.23123e2. But you can change that with 
different messages to the number printer.

```smalltalk
rws := ReadWriteStream with: String new.
fp := FloatPrinterScientificFormat new.
fp exponentDigitCount: 2.
fp exponentChar: $E.
fp print: 123.123 on: rws.
rws contents  "1.23123E02"
```

Sadly this is a bit more verbose than python: `'{:.5E}'.format(0.09112346) = '9.11235E-02'`
