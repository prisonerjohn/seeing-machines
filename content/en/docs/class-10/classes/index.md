---
title: "Classes"
description: ""
lead: ""
date: 2022-11-13T09:54:33-05:00
lastmod: 2022-11-13T09:54:33-05:00
draft: false
images: []
menu:
  docs:
    parent: "class-10"
    identifier: "classes"
weight: 1010
toc: true
---

As our applications grow in features and complexity, we need to organize our code a little better. We cannot just throw everything inside `ofApp` as it quickly becomes unwieldy and hard to navigate.

We can use classes to separate our app functionality into different modules. A class is a structure that contains variables and functions as members.

* A *class* is the definition of this structure.
* An *object* is an instance of the class.

For example, in the following code snippet, `ofPixels` is the class and `grabberPix` is the object.

```cpp
ofPixels grabberPix;
```

We have already been using classes throughout the course, as C++ is an object-oriented language. `ofPixels`, `ofTexture`, `ofxRealSense2::Device` are all classes. Even our application `ofApp` is a class and when we run our programs, we are instantiating an `ofApp` object and running it.

What we want to do now is try to create our own custom classes to hold our application code. While we can define our classes in any text file (as long as the compiler knows where to find it), there are conventions we should follow to keep things organized. Since we are writing code in OF, we will follow OF conventions:

* Classes should go in their own text files. One file for the header `.h` and another for the implementation `.cpp`.
* Either use a namespace or a prefix to identify what "group" this class belongs to. OF is slowly moving to namespaces, but for now all core classes are prefixed with "of".
* The class name should be capitalized, but the prefix should be all lowercase.

For example, the `ezFilter` class would be defined in the files `ezFilter.h` and `ezFilter.cpp`.

The compiler needs to know where to find all the class files needed by the project. The easiest way to do this is to add them to the `src` folder of your project. We can also put them in a subfolder of `src`, in which case we should re-run the Project Generator. This will find all subfolders and add them to the compiler *search paths*.

## Creating Classes

The process of creating OF classes is a little different in Visual Studio and Xcode, so we will go through both here.

### Visual Studio

1. Right-click on the `src` folder in the Solution Explorer, then navigate to "Add" -> "New Item..."
{{< image src="vs-addfile-01.png" alt="Visual Studio Add File" width="600px" >}}
1. In the dialog that pops up, select "C++ File" from the list and name your class in the "Name" field. Make sure the "Location" is set to the `src` folder in your project.
{{< image src="vs-addfile-02.png" alt="Visual Studio Add File" width="600px" >}}
1. Repeat these steps, this time selecting "Header File" from the list.
{{< image src="vs-addfile-03.png" alt="Visual Studio Add File" width="600px" >}}
1. We should then see new `.h` and `.cpp` files in the Solution Explorer. These are sometimes added to the root folder; if that's the case just drag them into the `src` group.
{{< image src="vs-addfile-04.png" alt="Visual Studio Add File" width="600px" >}}

The header file will have a single line of placeholder content in it.

```cpp
// ezBall.h
#pragma once
```

As a reminder, `#pragma once` is an [*include guard*](https://www.learncpp.com/cpp-tutorial/header-guards/) command, and its purpose is to make sure the class is only defined once.

* One way to understand this is to think that the compiler holds a table of contents of every class and its functions available in the application.
* Whenever it sees a header file, it adds its contents to this table of contents.
* In C++, we use an `#include` directive whenever we want to use a class defined in a different file. Whenever we use `#include`, the contents of that file are added to our table of contents.
* If we want to reuse the same class in many different places, we would include it again, and its contents would be repeated in our table of contents, causing a conflict!
* This is where include guards (or header guards) come in; they ensure that the contents of the header file are only included once.

### Xcode

1. Right-click on the `src` folder in the Project Navigator, then select"New File..."
{{< image src="xc-addfile-01.png" alt="Xcode Add File" width="600px" >}}
1. Select "C++ File" from the list.
{{< image src="xc-addfile-02.png" alt="Xcode Add File" width="600px" >}}
1. Name your class in the next dialog, and make sure "Also create header file" is selected.
{{< image src="xc-addfile-03.png" alt="Xcode Add File" width="600px" >}}
1. Ensure that the files will be saved to the `src` folder, and added to the `src` group in the next dialog.
{{< image src="xc-addfile-04.png" alt="Xcode Add File" width="600px" >}}
1. We should then see new `.hpp` and `.cpp` files in the Project Navigator.
{{< image src="xc-addfile-05.png" alt="Xcode Add File" width="600px" >}}

The header looks quite different from the VS version.

```cpp
// ezBall.hpp
#ifndef ezBall_hpp
#define ezBall_hpp

#include <stdio.h>

#endif // ezBall_hpp
```

```cpp
// ezBall.cpp
#include "ezBall.hpp"
```

`.hpp` is an alternate extension for C++ header files. It is not used in OF, so we will change it to `.h` to follow convention.

* This can be done directly in the Xcode Project Navigator.
* Note that we will also need to update the references to this file, like the `#include` directive in the `.cpp` file.

The placeholder uses a different type of include guard. Lines that begin with `#` are called *compiler directives*. We can think of them as a programming language for the compiler.

* `#ifndef` stands for "if not defined". If this condition is true, then everything between this line and `#endif` will be included.
* `#define` defines a new variable.
* Those lines can therefore be interpreted as: "If the variable `ezBall_hpp` does not exist, create it and include everything else in the file".
* We would then define the class above the `#endif`, to make sure it only gets included once.

For consistency's sake, we will replace this with the `#pragma once` header guard.

We can also remove the call to `#include <stdio.h>` as we do not need to explicitly include it.

## Class Members

We should now include a basic class definition in our files:

* The header will define and name the class, and include sections for all `public`  and `private` member variables and functions.
* The implementation will start with an `#include` directive, telling it where the class definiton is. This is simply the header file.

```cpp
// ezBall.h
#pragma once

class ezBall
{
public:

private:

};
```

```cpp
// ezBall.cpp
#include "ezBall.h"
```

Let's add basic functionality to our `ezBall`.

* We will use similar terminology to OF, adding a `setup()` function to initialize the object, and a `draw()` function to render it.
* We will add class variables for position, mass (which we'll use as radius), and color.
* We will include `ofMain.h` in our header file to give us access to openFrameworks objects and functions.

```cpp
// ezBall.h
#pragma once

#include "ofMain.h"

class ezBall
{
public:
  void setup(int x, int y);

  void draw();

private:
  glm::vec2 pos;

  float mass;

  ofColor color;
};
```

```cpp
// ezBall.cpp
#include "ezBall.h"

void ezBall::setup(int x, int y)
{
  pos = glm::vec2(x, y);
  mass = ofRandom(10, 30);
  color = ofColor(ofRandom(127, 255), ofRandom(127, 255), ofRandom(127, 255));
}

void ezBall::draw()
{
  ofSetColor(color);
  ofDrawCircle(pos, mass);
}
```

We can now include the `ezBall` header in our main app, and instantiate and use `ezBall` objects, calling any public members defined in the class.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ezBall.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  void keyPressed(int key);
  void mouseDragged(int x, int y, int button);
  void mousePressed(int x, int y, int button);

  void addBall(int x, int y);

  vector<ezBall> balls;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofBackground(0);
}

void ofApp::update()
{

}

void ofApp::draw()
{
  for (int i = 0; i < balls.size(); i++)
  {
    balls[i].draw();
  }
  // OR
  //for (auto b : balls)
  //{
  //  b.draw();
  //}
}

void ofApp::addBall(int x, int y)
{
  // Add a new ezBall.
  balls.push_back(ezBall());
  // Setup the last added ezBall.
  balls.back().setup(x, y);
}

void ofApp::keyPressed(int key)
{
  if (key == ' ')
  {
    balls.clear();
  }
}

void ofApp::mouseDragged(int x, int y, int button)
{
  addBall(x, y);
}

void ofApp::mousePressed(int x, int y, int button)
{
  addBall(x, y);
}
```

Let's add a bit of physics to make the `ezBall` more dynamic. We will add a new `update()` method to pass a force to the ball, and calculate its position every frame.

```cpp
// ezBall.h
#pragma once

#include "ofMain.h"

class ezBall
{
public:
  void setup(int x, int y);

  void update(glm::vec2 force);
  void draw();

private:
  glm::vec2 pos;
  glm::vec2 vel;
  glm::vec2 acc;

  float mass;

  ofColor color;
};
```

```cpp
// ezBall.cpp
#include "ezBall.h"

void ezBall::setup(int x, int y)
{
  pos = glm::vec2(x, y);
  mass = ofRandom(10, 30);
  color = ofColor(ofRandom(127, 255), ofRandom(127, 255), ofRandom(127, 255));
}


void ezBall::update(glm::vec2 force)
{
  // Add force.
  acc += force / mass;

  if (glm::length(vel) > 0)
  {
    // Add friction.
    glm::vec2 friction = glm::normalize(vel * -1) * 0.01f;
    acc += friction;
  }

  // Apply acceleration, then reset it!
  vel += acc;
  acc = glm::vec2(0.0f);

  // Move object.
  pos += vel;

  // Bounce off walls, taking radius into consideration.
  if (pos.x - mass < 0 || pos.x + mass > ofGetWidth())
  {
    pos.x = ofClamp(pos.x, mass, ofGetWidth() - mass);
    vel.x *= -1;
  }
  if (pos.y - mass < 0 || pos.y + mass > ofGetHeight())
  {
    pos.y = ofClamp(pos.y, mass, ofGetHeight() - mass);
    vel.y *= -1;
  }
}

void ezBall::draw()
{
  ofSetColor(color);
  ofDrawCircle(pos, mass);
}
```

As we add more functionality to the class, this becomes available to any object using `ezBall` objects.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofBackground(0);
}

void ofApp::update()
{
  glm::vec2 gravity = glm::vec2(0, 9.8f);

  for (int i = 0; i < balls.size(); i++)
  {
    balls[i].update(gravity);
  }
  // OR
  //for (auto b : balls)
  //{
  //  b.update(gravity);
  //}
}

void ofApp::draw()
{
  for (int i = 0; i < balls.size(); i++)
  {
      balls[i].draw();
  }
  // OR
  //for (auto b : balls)
  //{
  //  b.draw();
  //}
}

void ofApp::addBall(int x, int y)
{
  // Add a new ezBall.
  balls.push_back(ezBall());
  // Setup the last added ezBall.
  balls.back().setup(x, y);
}

void ofApp::keyPressed(int key)
{
  if (key == ' ')
  {
    balls.clear();
  }
}

void ofApp::mouseDragged(int x, int y, int button)
{
  addBall(x, y);
}

void ofApp::mousePressed(int x, int y, int button)
{
  addBall(x, y);
}
```

{{< image src="balls-bounce.png" alt="Balls Bounce" caption="*Balls Bounce*" width="600px" >}}
