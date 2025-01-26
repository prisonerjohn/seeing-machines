---
title: "Logging"
description: ""
lead: ""
date: 2022-10-07T11:52:44-04:00
lastmod: 2022-10-07T11:52:44-04:00
draft: true
images: []
menu:
  docs:
    parent: "class-5"
    identifier: "logging"
weight: 510
toc: true
---

openFrameworks has an advanced logging system that can be useful for reporting and debugging your applications.

## Log Level

Log messages have a "severity" level:

* `OF_LOG_VERBOSE`: For extra information (TMI)
* `OF_LOG_NOTICE`: For normal reporting, like status updates, state changes, etc.
* `OF_LOG_WARNING`: For minor errors that can be ignored, like an image that is not the dimensions you expected.
* `OF_LOG_ERROR`: For major errors that should be handled, like an image that fails to load because the file does not exist.
* `OF_LOG_FATAL_ERROR`: For showstopper errors, like not finding a camera for a vision based app.

Messages are output to the console using the [`ofLog(...)`](https://openframeworks.cc/documentation/utils/ofLog/#show_ofLog) method and passing a log level.

```cpp
ofLog(OF_LOG_NOTICE) << "Hey this is a message!";
```

Note the use of the left-shift operator `<<`. This is similar to how we have been logging messages using `cout` up to now.

This special `<<` operator tends to be used when we are "pushing" or "adding" something to something else. In this case, we are pushing `string` objects to the logger, which writes them all out in a single line.

The "something" we are pushing must either be a `string` or an object that the compiler knows how to convert to a `string`. It already knows how to handle numbers (`int` and `float`) and simple objects (`ofPoint` and `ofRectangle`) but for anything custom or more complex, we will need to break that out ourselves.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofLog(OF_LOG_VERBOSE) << "The app is now starting!";

  if (image.load("dog-grass.jpg"))
  {
    float imageRatio = image.getWidth() / image.getHeight();
    if (imageRatio != 16.0 / 9.0)
    {
      ofLog(OF_LOG_WARNING) << "Dog image has the wrong aspect ratio " << imageRatio << ", it will be stretched!";
    }
    else
    {
      ofLog(OF_LOG_NOTICE) << "Dog image loaded successfully with dimensions " << image.getWidth() << "x" << image.getHeight() << ".";
    }
  }
  else
  {
    ofLog(OF_LOG_ERROR) << "Unable to load dog image, make sure it's in the data folder!";
  }

  if (!grabber.setup(1280, 720))
  {
    ofLog(OF_LOG_FATAL_ERROR) << "Unable to open camera, there's no reason to keep going :(";
  }
}
```

Notice that all messages are output to the console except the first VERBOSE message.

```cpp
[ error ] ofImage: loadImage(): couldn't load image from ""dog-grass.jpg""
[ error ] Unable to load dog image, make sure it's in the data folder!
[ fatal ] Unable to open camera, there's no reason to keep going :(
```

This is because the default log level is `OF_LOG_NOTICE`. Any log messages using a level above this will not be printed. This can be changed with a call to [`ofSetLogLevel()`](https://openframeworks.cc/documentation/utils/ofLog/#show_ofSetLogLevel). The parameter is any of the log levels listed above, or `OF_LOG_SILENT` which will disable all logging no matter what the severity is.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  //ofSetLogLevel(OF_LOG_VERBOSE);
  //ofSetLogLevel(OF_LOG_NOTICE);  // default
  ofSetLogLevel(OF_LOG_WARNING);
  //ofSetLogLevel(OF_LOG_ERROR);
  //ofSetLogLevel(OF_LOG_FATAL_ERROR);
  //ofSetLogLevel(OF_LOG_SILENT);

  ofLog(OF_LOG_VERBOSE) << "The app is now starting!";

  if (image.load("dog-grass.jpg"))
  {
    float imageRatio = image.getWidth() / image.getHeight();
    if (imageRatio != 16.0 / 9.0)
    {
      ofLog(OF_LOG_WARNING) << "Dog image has the wrong aspect ratio " << imageRatio << ", it will be stretched!";
    }
    else
    {
      ofLog(OF_LOG_NOTICE) << "Dog image loaded successfully with dimensions " << image.getWidth() << "x" << image.getHeight() << ".";
    }
  }
  else
  {
    ofLog(OF_LOG_ERROR) << "Unable to load dog image, make sure it's in the data folder!";
  }

  if (!grabber.setup(1280, 720))
  {
    ofLog(OF_LOG_FATAL_ERROR) << "Unable to open camera, there's no reason to keep going :(";
  }
}
```

We would change the log level depending on if we are debugging our app, running it for a demo, or preparing a build for release. Instead of having to go through all our code and commenting or deleting all the log lines, a simple call to `ofSetLogLevel()` can set the appropriate level of console logging.

## Log Modules

The `ofLog()` methods can be swapped out for their `ofLogXXX()` counterpart. So `ofLog(OF_LOG_NOTICE)` becomes `ofLogNotice()`, `ofLog(OF_LOG_ERROR)` becomes `ofLogError()` and so on, which is a little less verbose.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  //ofSetLogLevel(OF_LOG_VERBOSE);
  //ofSetLogLevel(OF_LOG_NOTICE);  // default
  ofSetLogLevel(OF_LOG_WARNING);
  //ofSetLogLevel(OF_LOG_ERROR);
  //ofSetLogLevel(OF_LOG_FATAL_ERROR);
  //ofSetLogLevel(OF_LOG_SILENT);

  ofLogVerbose() << "The app is now starting!";

  if (image.load("dog-grass.jpg"))
  {
    float imageRatio = image.getWidth() / image.getHeight();
    if (imageRatio != 16.0 / 9.0)
    {
      ofLogWarning() << "Dog image has the wrong aspect ratio " << imageRatio << ", it will be stretched!";
    }
    else
    {
      ofLogNotice() << "Dog image loaded successfully with dimensions " << image.getWidth() << "x" << image.getHeight() << ".";
    }
  }
  else
  {
    ofLogError() << "Unable to load dog image, make sure it's in the data folder!";
  }

  if (!grabber.setup(1280, 720))
  {
    ofLogFatalError() << "Unable to open camera, there's no reason to keep going :(";
  }
}
```

Another advantage of this version is that we can add an optional module as a parameter to the function. The module is just a string that can represent the part / section of the code we are dealing with.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  //ofSetLogLevel(OF_LOG_VERBOSE);
  //ofSetLogLevel(OF_LOG_NOTICE);  // default
  ofSetLogLevel(OF_LOG_WARNING);
  //ofSetLogLevel(OF_LOG_ERROR);
  //ofSetLogLevel(OF_LOG_FATAL_ERROR);
  //ofSetLogLevel(OF_LOG_SILENT);

  ofLogVerbose("App") << "The app is now starting!";

  if (image.load("dog-grass.jpg"))
  {
    float imageRatio = image.getWidth() / image.getHeight();
    if (imageRatio != 16.0 / 9.0)
    {
      ofLogWarning("Image Load") << "Dog image has the wrong aspect ratio " << imageRatio << ", it will be stretched!";
    }
    else
    {
      ofLogNotice("Image Load") << "Dog image loaded successfully with dimensions " << image.getWidth() << "x" << image.getHeight() << ".";
    }
  }
  else
  {
    ofLogError("Image Load") << "Unable to load dog image, make sure it's in the data folder!";
  }

  if (!grabber.setup(1280, 720))
  {
    ofLogFatalError("Grabber Init") << "Unable to open camera, there's no reason to keep going :(";
  }
}
```

The module will prepend your log message in the output.

```cpp
[ error ] ofImage: loadImage(): couldn't load image from ""dog-grass.jpg""
[ error ] Image Load: Unable to load dog image, make sure it's in the data folder!
[ fatal ] Grabber Setup: Unable to open camera, there's no reason to keep going :(
```

Instead of coming up with a module name every time, we can also use the `__FUNCTION__` macro, which will automatically get replaced by the full method name during compilation.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  //ofSetLogLevel(OF_LOG_VERBOSE);
  //ofSetLogLevel(OF_LOG_NOTICE);  // default
  ofSetLogLevel(OF_LOG_WARNING);
  //ofSetLogLevel(OF_LOG_ERROR);
  //ofSetLogLevel(OF_LOG_FATAL_ERROR);
  //ofSetLogLevel(OF_LOG_SILENT);

  ofLogVerbose(__FUNCTION__) << "The app is now starting!";

  if (image.load("dog-grass.jpg"))
  {
    float imageRatio = image.getWidth() / image.getHeight();
    if (imageRatio != 16.0 / 9.0)
    {
      ofLogWarning(__FUNCTION__) << "Dog image has the wrong aspect ratio " << imageRatio << ", it will be stretched!";
    }
    else
    {
      ofLogNotice(__FUNCTION__) << "Dog image loaded successfully with dimensions " << image.getWidth() << "x" << image.getHeight() << ".";
    }
  }
  else
  {
    ofLogError(__FUNCTION__) << "Unable to load dog image, make sure it's in the data folder!";
  }

  if (!grabber.setup(1280, 720))
  {
    ofLogFatalError(__FUNCTION__) << "Unable to open camera, there's no reason to keep going :(";
  }
}
```

```cpp
[ error ] ofImage: loadImage(): couldn't load image from ""dog-grass.jpg""
[ error ] ofApp::setup: Unable to load dog image, make sure it's in the data folder!
[ fatal ] ofApp::setup: Unable to open camera, there's no reason to keep going :(
```

## Log Channels

`ofLog()` does not necessarily need to log to the console. It might make more sense to log to a file that can be examined later, especially in the context of an installation going live. This can be done using the function [`ofLogToFile()`](https://openframeworks.cc/documentation/utils/ofLog/#show_ofLogToFile).

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  //ofSetLogLevel(OF_LOG_VERBOSE);
  //ofSetLogLevel(OF_LOG_NOTICE);  // default
  ofSetLogLevel(OF_LOG_WARNING);
  //ofSetLogLevel(OF_LOG_ERROR);
  //ofSetLogLevel(OF_LOG_FATAL_ERROR);
  //ofSetLogLevel(OF_LOG_SILENT);

  ofLogToFile("log-" + ofGetTimestampString() + ".txt", true);

  ofLogVerbose(__FUNCTION__) << "The app is now starting!";

  if (image.load("dog-grass.jpg"))
  {
    float imageRatio = image.getWidth() / image.getHeight();
    if (imageRatio != 16.0 / 9.0)
    {
      ofLogWarning(__FUNCTION__) << "Dog image has the wrong aspect ratio " << imageRatio << ", it will be stretched!";
    }
    else
    {
      ofLogNotice(__FUNCTION__) << "Dog image loaded successfully with dimensions " << image.getWidth() << "x" << image.getHeight() << ".";
    }
  }
  else
  {
    ofLogError(__FUNCTION__) << "Unable to load dog image, make sure it's in the data folder!";
  }

  if (!grabber.setup(1280, 720))
  {
    ofLogFatalError(__FUNCTION__) << "Unable to open camera, there's no reason to keep going :(";
  }
}
```

We can switch back to console logging with [`ofLogToConsole()`](https://openframeworks.cc/documentation/utils/ofLog/#show_ofLogToConsole).
