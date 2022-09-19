---
title: "Variables and Arrays"
description: ""
lead: ""
date: 2022-09-18T13:41:13-04:00
lastmod: 2022-09-18T13:41:13-04:00
draft: false
images: []
menu:
  docs:
    parent: "class-1"
    identifier: "variables-and-arrays"
weight: 210
toc: true
---

## Data Types

Let's start with the basics and review data types in C++.

### Main Primitives

#### `int`

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

#### `unsigned int`

* the unsigned (no +/- sign) version of `int`, only positive numbers
* represents `0` and positive whole numbers up to `4,294,967,295`

#### `char`

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

#### `unsigned char`

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

#### `bool`

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

#### `float`

* 32 bits of data
* represents a decimal number with ~7 significant digits
* `float` is short for "floating point", which means that the decimal point can move positions (e.g. we can represent `1.23456` and `123.456` with the same amount of memory)

```cpp
float videoWidth = 1920;
float videoHeight = 1080;
float aspectRatio = videoWidth / videoHeight;  // 1.777778
```

### Additional Primitives

The following primitive types are not used as often but are still useful if we need to optimize and use less memory, or increase precision and use more memory.

#### `short` and `unsigned short`

* 16 bits of data
* represents whole numbers between `-32,768` and `32,767` (signed) or `0` and `65,535` (unsigned)

#### `long` and `unsigned long`

* 64 bits of data
* represents whole numbers between `-9M` and `9M` (signed) or `0` and `18M` (unsigned)

#### `double`

* 64 bits of data
* represents floating point numbers with ~15 significant digits

### Strings

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

#### Concatenation

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

## Arrays

Arrays are used to store multiple values of the same type under a single variable (vs declaring one variable per value).

```cpp
// An array containing 10 integers (uninitialized).
int values[10];
```

Array elements are accessed using the square bracket `[]` operator. We can pass in an index to access the corresponding element in the array. Note that arrays are `0`-indexed (the index of the first element is `0`) in C++.

```cpp
int values[10];
for (int i = 0; i < 10; i++)
{
  values[i] = i + 1;
}
```

{{< image src="array-index.png" alt="Array indices" align="center" >}}

### Strings as Arrays

A `string` is an array of `char` under the hood (along with some extra functionality). Each character in a `string` is an element in the array and can be accessed using the `[]` notation.

{{< details "How would we print out the value of a string one character at a time, using array notation?" >}}

```cpp
string name = "John Doe";
cout << "The name '";
for (int i = 0; i < name.size(); i++)
{
  cout << name[i];
}
cout << "' has " << name.size() << " characters" << endl;
```

{{< image src="array-name.png" alt="String as array" align="center" >}}

{{< /details >}}

### 2D Arrays

Arrays of other arrays are called multidimensional arrays. Instead of each array position holding a single element, it holds an entire array of elements.

Although arrays can have any number of dimensions, we will most often work with two-dimensional arrays.

```cpp
// An array containing 10 arrays each containing 2 integers (uninitialized).
int values[10][2];
```

Array elements are accessed using multiple square brackets `[][]` (one per dimension). Nested for-loops can be used to access all the elements.

```cpp
int values[10][2];  // 10 columns by 2 rows
for (int y = 0; y < 2; y++)
{
  for (int x = 0; x < 10; x++)
  {
    values[x][y] = x + y;
  }
}
```

{{< image src="array-2d.png" alt="2D array" align="center" >}}

{{< details "How would we set each value to a sequential index (from 0 to 19)?" >}}

We need to consider each dimension separately as columns and rows, and look at the array as "columns of rows". To access a row index (`0-1`), we always need to add the column offset first (`0-9`). Each column has `10` rows, so the offset must be multiplied by 10.

```cpp
int values[10][2];  // 10 columns by 2 rows
for (int y = 0; y < 2; y++)
{
  for (int x = 0; x < 10; x++)
  {
    values[x][y] = y * 10 + x;
  }
}
```

{{< image src="array-2d-idx.png" alt="2D array index" align="center" >}}

{{< /details >}}

{{< details "How would we fill an array of 40 columns by 30 columns with a random <code>true</code> or <code>false</code> value? How could we print it out to the console as a grid layout?" >}}

We can use the [`ofRandomuf()`](https://openframeworks.cc/documentation/math/ofMath/#!show_ofRandomuf) OF function, which returns a random value between `0` and `1` (the "uf" stands for unsigned float). We will set our element to `false` if the random value is less than `0.5`, and set it to `true` if the value is greater than `0.5`.

```cpp
bool values[40][30];

// Fill the 2D array with 0s and 1s.
for (int y = 0; y < 30; y++)  // rows
{
  for (int x = 0; x < 40; x++)  // columns
  {
    if (ofRandomuf() < 0.5)
    {
        values[x][y] = false;
    }
    else
    {
        values[x][y] = true;
    }
  }
}
```

To output the values as a grid, we can once again use nested for-loops to go through the 2D array, print out the characters one at a time, and print out a new line whenever we increment the row index.

```cpp
// Read back the values as a grid.
for (int y = 0; y < 30; y++)  // rows
{
  for (int x = 0; x < 40; x++)  // columns
  {
    cout << values[x][y];
  }
  cout << endl;  // Add a new line after every row.
}
```

{{< /details >}}

{{< details "How would we visualize this grid as pixels on screen?" >}}

We can draw a cell for each grid position using [`ofDrawRectangle(...)`](https://openframeworks.cc/documentation/graphics/ofGraphics/#!show_ofDrawRectangle), setting the color for each cell in the loop using [`ofSetColor(...)`](https://openframeworks.cc/documentation/graphics/ofGraphics/#show_ofSetColor).

```cpp
// Draw the array as a grid.
int gridSize = 10;
for (int y = 0; y < 30; y++)  // rows
{
  for (int x = 0; x < 40; x++)  // columns
  {
    if (values[x][y])
    {
      ofSetColor(255);
    }
    else
    {
      ofSetColor(0);
    }
    ofDrawRectangle(x * gridSize, y * gridSize, gridSize, gridSize);
  }
}
```

{{< /details >}}

{{< details "How would we only set the edge values (border pixels) to <code>true</code> and the remaining pixels to <code>false</code>?<p></p><div style='text-align:center;margin:auto;'><img src='array-border.png'></div>" >}}

* The left edge elements have their column index set to `0`.
* The top edge elements have their row index set to `0`.
* The right edge elements have their column index set to the number of columns (the width) minus 1. In our case, this is `39`.
* The bottom edge elements have their row index set to the number of rows (the height) minus 1. In our case, this is `29`.

If any of the above conditions are `true`, our value should also be `true`.

```cpp
for (int y = 0; y < 30; y++)  // rows
{
  for (int x = 0; x < 40; x++)  // columns
  {
    if (x == 0 || y == 0 || x == 39 || y == 29)
    {
      values[x][y] = true;
    }
    else
    {
      values[x][y] = false;
    }
    
    // The following does the same thing as a single line of code.
    //values[x][y] = (x == 0 || y == 0 || x == 39 || y == 29);
  }
}
```

{{< /details >}}

{{< details "How would we create a grid pattern where each cell is 10x10 units?<p></p><div style='text-align:center;margin:auto;'><img src='array-grid.png'></div>" >}}

We want every 10th element in each direction to be `true`, and the remaining values to be `false`. We can use the modulo `%` operator, which is ideal for whenever we want to count things periodically (e.g. every X count, do something).

```cpp
for (int y = 0; y < 30; y++)  // rows
{
  for (int x = 0; x < 40; x++)  // columns
  {
    if (x % 10 == 0 || y % 10 == 0)
    {
      values[x][y] = true;
    }
    else
    {
      values[x][y] = false;
    }
    
    // The following does the same thing as a single line of code.
    //values[x][y] = (x % 10 == 0 || y % 10 == 0);
  }
}
```

We do not end up with a border on the right or the bottom because `39` and `29` are not divisible by `10`. If this is something we wanted, one option would be to increase our 2D array size to 41 by 31, making the last column index `40` and the last row index `30`.
{{< /details >}}

### 1D to 2D Interpretation

There is no difference in computer between a 1D and a 2D array, they are both just many indexed elements in sequence.

We could re-write some of our previous examples using a one-dimensional array, but using two-dimensional access.

```cpp
bool values[40*30];

// Fill the 2D array with 0s and 1s.
for (int y = 0; y < 30; y++)  // rows
{
  for (int x = 0; x < 40; x++)  // columns
  {
    // Calculate the index using the column value, the row value, and the row offset.
    int idx = y * 40 + x;
    if (x == 0 || y == 0 || x == 39 || y == 29)
    {
        values[idx] = true;
    }
    else
    {
        values[idx] = false;
    }

    // The following does the same thing as a single line of code.
    //values[idx] = (x == 0 || y == 0 || x == 39 || y == 29);
  }
}
```

In the last few examples, we have been using arrays to generate images. We have interpreted the array element values as colors. This is, in fact, how images are usually stored in computer memory. We will explore this further in the next section.
