---
title: "Pointers"
description: ""
lead: ""
date: 2022-10-23T14:19:26-04:00
lastmod: 2022-10-23T14:19:26-04:00
draft: false
images: []
menu:
  docs:
    parent: "class-5"
    identifier: "pointers"
weight: 610
toc: true
---

So far in this course, every time we have wanted to reference some data, we have done so directly using variables.

```cpp
ofImage dogImg;
```

`dogImg` is a variable that can hold an `ofImage`. The variable is referencing the object itself, and we can access the variables and functions inside of it using the dot `.` operator.

```cpp
dogImg.load("dog-grass.jpg");
dogImg.draw(0, 0);
```

## Operators

In C++, we can also reference an object using a *pointer* variable. A pointer does not reference the object itself, but the address in memory where the object is stored.

Pointers have a special notation. To create a pointer variable, we need to add a star `*` right after the type when declaring the variable.

```cpp
ofImage* dogImgPtr;
```

{{< alert context="info" icon="✌️" >}}
**Star Position**

The star `*` can be positioned right after the variable type, right before the variable name, or by itself between the variable type and name.

The following are all equivalent and it is up to the programmer to determine what style they want to adhere to.

```cpp
ofImage* dogImgPtr;  // OK
ofImage *dogImgPtr;  // OK
ofImage * dogImgPtr; // OK
```

{{< /alert >}}

Reference variables are automatically allocated and created as soon as they are declared. All variables we have put in our `ofApp` header so far have been created right as the program started running.

Pointers on the other hand are not automatically created. They must be explicitly *instantiated* using the `new` operator.

```cpp
dogImgPtr = new ofImage();
```

Pointers are also not automatically deleted. They must be explicitly *destroyed* using the `delete` operator when we are done with them. As a general rule, anything we create with `new`, we should eventually destroy with `delete` down the line.

```cpp
delete dogImgPtr;
```

To access the variables and functions contained inside an object referenced by a pointer, we have to use the arrow `->` operator.

```cpp
dogImgPtr->load("dog-grass.jpg");
dogImgPtr->draw(0, 0);
```

We can convert a reference to a pointer, this is called *address-of* . Putting an ampersand `&` operator in front of a reference variable will return its address in memory.

```cpp
ofImage dogImg;
ofImage* dogImgPtr = &dogImg;
```

Conversely, we can access the variable referenced by a pointer directly, which is called *dereferencing*. Putting a star `*` in front of a pointer variable will return the object it references.

```cpp
ofImage* dogImgPtr = new ofImage();
ofImage dogImg = *dogImgPtr;
```

These operators are useful for passing variables to functions, when we are working with pointers but the function expects a reference or vice-versa.

## Arrays

Arrays have a close relationship with pointers. In fact, these are interchangeable in most cases.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

class ofApp : public ofBaseApp 
{
public:
  void setup();
  void update();
  void draw();

  ofVideoGrabber grabber;
  ofImage resultImg;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640 * 2, 480);

  grabber.setup(640, 480);
  resultImg.allocate(640, 480, OF_IMAGE_GRAYSCALE);
}

void ofApp::update()
{
  grabber.update();

  int threshold = 127;

  unsigned char* grabberData = grabber.getPixels().getData();
  unsigned char* resultData = resultImg.getPixels().getData();
  for (int i = 0; i < grabber.getWidth() * grabber.getHeight(); i++)
  {
    int r = grabberData[i * 3 + 0];
    int g = grabberData[i * 3 + 1];
    int b = grabberData[i * 3 + 2];

    if (r > threshold && g > threshold && b > threshold)
    {
      resultData[i] = 255;
    }
    else
    {
      resultData[i] = 0;
    }
  }

  resultImg.update();
}

void ofApp::draw()
{
  grabber.draw(0, 0, 640, 480);
  resultImg.draw(640, 0, 640, 480);
}
```

An array reference is just a pointer to its first element. We can use pointer arithmetic to move this reference, and even to iterate through the array!

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640 * 2, 480);

  grabber.setup(640, 480);
  resultImg.allocate(640, 480, OF_IMAGE_GRAYSCALE);
}

void ofApp::update()
{
  grabber.update();

  int threshold = 127;

  unsigned char* grabberPtr = grabber.getPixels().getData();
  unsigned char* resultPtr = resultImg.getPixels().getData();
  for (int i = 0; i < grabber.getWidth() * grabber.getHeight(); i++)
  {
    int r = *(grabberPtr++);
    int g = *(grabberPtr++);
    int b = *(grabberPtr++);

    if (r > threshold && g > threshold && b > threshold)
    {
      *resultPtr = 255;
    }
    else
    {
      *resultPtr = 0;
    }
    resultPtr++;
  }

  resultImg.update();
}

void ofApp::draw()
{
  grabber.draw(0, 0, 640, 480);
  resultImg.draw(640, 0, 640, 480);
}
```

## Pros

More control over object creation.

* We can choose *when* to create an object. For example, create a set of `ofImage` objects every time a new camera is plugged into the computer.
* We can choose *how* to create an object. Reference variables will use the default object *constructor* but pointer variables can use any available constructor.

```cpp
// Create an ofImage and load a file in one line.
ofImage* dogImgPtr = new ofImage("dog-grass.jpg");
```

Simple variable passing.

* We have already seen that variables are passed by value by default. Whenever we use the assignment operator `=` or pass variables to a function, we are making copies of those variables. In some situations, we can pass the variable by reference with the `&` operator, but this is not always supported.
* When manipulating pointers, we are always just working with memory addresses, so we never inadvertently make copies of objects.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

class ofApp : public ofBaseApp 
{
public:
  void setup();
  void draw();

  bool loadImageVal(ofImage img, string file);
  bool loadImageRef(ofImage& img, string file);
  bool loadImagePtr(ofImage* img, string file);

  ofImage dogImg;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  loadImageVal(dogImg, "dog-grass.jpg");
  //loadImageRef(dogImg, "dog-grass.jpg");
  //loadImagePtr(&dogImg, "dog-grass.jpg");
}

void ofApp::draw()
{
  dogImg.draw(0, 0);
}

bool ofApp::loadImageVal(ofImage img, string file)
{
  return img.load(file);
}

bool ofApp::loadImageRef(ofImage& img, string file)
{
  return img.load(file);
}

bool ofApp::loadImagePtr(ofImage* img, string file)
{
  return img->load(file);
}
```

## Cons

Memory management is up to the programmer.

* It is up to us to make sure a pointer is referencing a valid object when using it. If we try to access an uninitialized pointer, our app will either crash or we will corrupt memory in other parts of the app.
* It is up to us to properly deallocate the memory when we are done. If we re-allocate a pointer before deleting its previous contents, we end up with a chunk of memory that is reserved but inaccessible. This is called a *memory leak*.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

class ofApp : public ofBaseApp 
{
public:
  void setup();
  void draw();
  void exit();

  bool loadImageVal(ofImage img, string file);
  bool loadImageRef(ofImage& img, string file);
  bool loadImagePtr(ofImage* img, string file);

  ofImage* dogImgPtr;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  //dogImgPtr = new ofImage();
  //loadImageVal(*dogImgPtr, "dog-grass.jpg");
  //loadImageRef(*dogImgPtr, "dog-grass.jpg");
  loadImagePtr(dogImgPtr, "dog-grass.jpg");
}

void ofApp::exit()
{
  delete dogImgPtr;
}

void ofApp::draw()
{
  dogImgPtr->draw(0, 0);
}

bool ofApp::loadImageVal(ofImage img, string file)
{
  return img.load(file);
}

bool ofApp::loadImageRef(ofImage& img, string file)
{
  return img.load(file);
}

bool ofApp::loadImagePtr(ofImage* img, string file)
{
  return img->load(file);
}
```

{{< alert context="info" icon="✌️" >}}
**The exit() function in ofApp**

The example above includes a new built-in `ofApp` method called [`exit()`](https://openframeworks.cc/documentation/application/ofBaseApp/#!show_exit). As the name implies, this function is called right before the app quits. It is like a counterpart to the `setup()` function and is only called once.

This is a good place to do any cleanup before the app closes completely.
{{< /alert >}}

* It is also up to us to make sure our pointer references are valid, which can become tricky when we have multiple pointers referencing the same object.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

class ofApp : public ofBaseApp 
{
public:
  void setup();
  void draw();
  void exit();

  void mousePressed(int x, int y, int button);

  bool loadImagePtr(ofImage* img, string file);

  ofImage* dogImgPtrLoad;
  ofImage* dogImgPtrDraw;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  dogImgPtrLoad = new ofImage();
  loadImagePtr(dogImgPtrLoad, "dog-grass.jpg");
  
  dogImgPtrDraw = dogImgPtrLoad;
}

void ofApp::exit()
{
  delete dogImgPtrLoad;
  dogImgPtrLoad = nullptr;
}

void ofApp::draw()
{
  if (dogImgPtrDraw != nullptr)
  {
    dogImgPtrDraw->draw(0, 0);
  }
}

void ofApp::mousePressed(int x, int y, int button)
{
  delete dogImgPtrLoad;
  dogImgPtrLoad = nullptr;
}

bool ofApp::loadImagePtr(ofImage* img, string file)
{
  return img->load(file);
}
```

## Shared Pointers

Shared pointers are a sort of middle ground between references and pointers. They are actually wrappers around pointers that handle all the memory management and reference counting for us, essentially getting rid of the cons listed above. This comes at a performance cost, but it will be negligeable in most cases.

Shared pointers belong to a family of smart pointers, but we will only focus on shared pointers as they are the most common.

In a nutshell, they work the following way:

* A shared pointer is a templated type belonging to the `std` namespace: [`std::shared_ptr<T>`](http://www.cplusplus.com/reference/memory/shared_ptr/).
* A pointer is created using [`std::make_shared<>()`](http://www.cplusplus.com/reference/memory/make_shared/) or by assigning another pointer to it.
* A pointer is destroyed using [`std::shared_ptr<T>.reset()`](http://www.cplusplus.com/reference/memory/shared_ptr/reset/) or by simply setting it to `nullptr`.
* Every time a new pointer to the same object is added, the reference count increases.
* Every time a pointer to that object is removed, the reference count decreases.
* When the reference count hits 0, the object is destroyed.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

class ofApp : public ofBaseApp 
{
public:
  void setup();
  void draw();

  void mousePressed(int x, int y, int button);

  bool loadImagePtr(std::shared_ptr<ofImage> img, string file);

  shared_ptr<ofImage> dogImgPtrLoad;
  shared_ptr<ofImage> dogImgPtrDraw;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  dogImgPtrLoad = std::make_shared<ofImage>();
  loadImagePtr(dogImgPtrLoad, "dog-grass.jpg");

  dogImgPtrDraw = dogImgPtrLoad;
}

void ofApp::draw()
{
  if (dogImgPtrDraw)
  {
    dogImgPtrDraw->draw(0, 0);
  }
}

void ofApp::mousePressed(int x, int y, int button)
{
  if (button == 0)
  {
      dogImgPtrLoad.reset();
  }
  else
  {
      dogImgPtrDraw.reset();
  }
}

bool ofApp::loadImagePtr(std::shared_ptr<ofImage> img, string file)
{
  return img->load(file);
}
```
