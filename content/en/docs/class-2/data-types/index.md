---
title: "Data Types"
description: ""
lead: ""
date: 2022-09-18T13:41:13-04:00
lastmod: 2025-02-02T12:31:13-04:00
draft: false
images: []
menu:
  docs:
    parent: "class-2"
    identifier: "data-types"
weight: 210
toc: true
---

Let's start with the basics and review data types in C++.

## Main Primitives

### `int`

* 32 bits of data (usually but not always)
* represents a whole number between `-2,147,483,648` and `2,147,483,647`

```cpp
int videoWidth = 1920;
int videoHeight = 1080;
int numVideoPixels = videoWidth * videoHeight;
```

* integers do not support decimal points

{{< alert context="info" icon="✌️" >}}
**Integer division <code>/</code> and modulo <code>%</code> operators**

Operations on integers return integers. This is particularly important to remember with division `/`.

```cpp
int numStudents = 17;
int studentsPerTeam = 4;
int numTeams = numStudents / studentsPerTeam;  // 4 not 4.25!
```

The modulo `%` operator is used on integers to get the remainder of a division.

```cpp
int leftoverStudents = numStudents % studentsPerTeam;  // 1 student without a team :(
```

{{< /alert >}}

### `unsigned int`

* the unsigned (no +/- sign) version of `int`, only positive numbers
* represents `0` and positive whole numbers up to `4,294,967,295`

### `char`

* 8 bits of data
* represents a whole number between `-128` and `127`

```cpp
char numStudents = 17;
char studentsPerTeam = 4;
char numTeams = numStudents / studentsPerTeam;  // 4 not 4.25!
```

* if we try to use a `char` to represent a larger number, we will end up with the wrong value but C++ will not flag an error!

```cpp
char videoWidth = 1920;  // ?
```

### `unsigned char`

* the unsigned (no +/- sign) version of `char`, only positive numbers
* represents `0` and positive whole numbers up to `255`
* often used to represent characters in a string, using the [ASCII table](https://www.asciitable.com/) for conversion

```cpp
unsigned char H = 72;
unsigned char i = 'i';
cout << H << i << endl;
```

{{< alert context="info" icon="✌️" >}}
**What do <code>cout</code> and <code>endl</code> do?**

`cout` is a command to send text output to the console. The `<<` (left shift) operator is used to send data to the output, and can be used multiple times to add more text to the output.

New lines are not automatically added. The `endl` command is used to send a new line. This will usually be found at the end of a `cout` line of code.
{{< /alert >}}

### `bool`

* 1 bit of data
* represents `true` or `false`, `0` or `1`, "yes" or "no", etc.
* we can use the keywords `true` and `false` to set a boolean value

```cpp
bool isTheSkyBlue = true;
bool isThisBoring = false;
```

* we can also use numbers, where `0` evaluates to `false` and any other number evaluates to `true`

```cpp
bool numStudents = 17;  // true
if (numStudents)
{
  cout << "Class is in session!" << endl;
}
```

{{< alert context="danger" icon="⚠️" >}}
Note that even though you can use numbers to represent a boolean, the `bool` data type only has enough memory to represent `0` or `1`.

```cpp
bool numStudents = 17;
cout << "There are " << numStudents << " students in class" << endl;  // 1
```

{{< /alert >}}

### `float`

* 32 bits of data
* represents a decimal number with ~7 significant digits
* `float` is short for "floating point", which means that the decimal point can move positions (e.g. we can represent `1.23456` and `123.456` with the same amount of memory)

```cpp
float videoWidth = 1920;
float videoHeight = 1080;
float aspectRatio = videoWidth / videoHeight;  // 1.777778
```

## Additional Primitives

The following primitive types are not used as often but are still useful if we need to optimize and use less memory, or increase precision and use more memory.

### `short` and `unsigned short`

* 16 bits of data
* represents whole numbers between `-32,768` and `32,767` (signed) or `0` and `65,535` (unsigned)

### `long` and `unsigned long`

* 64 bits of data
* represents whole numbers between `-9M` and `9M` (signed) or `0` and `18M` (unsigned)

### `double`

* 64 bits of data
* represents floating point numbers with ~15 significant digits

## Strings

Like in most programming languages, strings (sequences of characters) are a complex class type, but they have special rules and optimizations applied to them since they are used very often.

```cpp
string name = "John Doe";
```

`string` objects have a variety of [methods](https://cplusplus.com/reference/string/string/) (class functions) we can use to access their properties.

```cpp
string name;
if (name.empty())
{
  cout << "No name has been set, using default..." << endl;
  name = "John Doe";
}
cout << "Hello, " << name << endl;
```

We can iterate through a `string` to get the characters it is made up of.

```cpp
string name = "John Doe";
cout << "Your name is: ";
for (int i = 0; i < name.size(); i++)
{
  cout << name.at(i);
}
cout << endl;
```

### Concatenation

`string` objects can be concatenated using the `+` operator.

```cpp
string first = "John";
string last = "Doe";
string name = first + " " + last;
```

This also sometimes works with non-`string` types, but not always as the compiler might not know how to use the `+` operator.

```cpp
int videoWidth = 1920;
int videoHeight = 1080;
string resolution = videoWidth + "x" + videoHeight;  // OK?

float videoWidthf = 1920;
float videoHeightf = 1080;
string resolutionf = videoWidthf + "x" + videoHeightf;  // ERROR!
```

OF has [`ofToString()`](https://openframeworks.cc/documentation/utils/ofUtils/#!show_ofToString) helper functions, which can be used to convert other variable types into `string`.

```cpp
int videoWidth = 1920;
int videoHeight = 1080;
string resolution = ofToString(videoWidth) + "x" + ofToString(videoHeight);

float videoWidthf = 1920;
float videoHeightf = 1080;
string resolutionf = ofToString(videoWidthf, 1) + "x" + ofToString(videoHeightf, 1);
```
