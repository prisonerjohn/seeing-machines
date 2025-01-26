---
title: "Mobile Development"
description: ""
lead: ""
date: 2022-11-19T15:25:33-05:00
lastmod: 2022-11-19T15:25:33-05:00
draft: true
images: []
menu:
  docs:
    parent: "class-11"
    identifier: "mobile-development"
weight: 1110
toc: true
---

One of the strengths of openFrameworks is that it runs on a multitude of platforms. We can usually run our apps on Android or iOS with minimal modifications.

## iOS

OF runs on most iOS platforms, including iPhones, iPads, and Apple TVs.

### Requirements

* A Mac and [Xcode](https://apps.apple.com/us/app/xcode/id497799835?mt=12) to compile apps for iOS.
* Xcode will ask to download additional iOS files if required, but should already include all necessary libraries and applications.
* An Apple ID. We can build apps for a local device (plugged into the computer) for free. However, if we want to distribute apps (ad-hoc or on the App Store), we will need a $99 Apple Developer account.
* The iOS version of openFrameworks. The following code has been tested with [version 0.12.0](https://github.com/openframeworks/openFrameworks/releases/download/0.12.0/of_v0.12.0_ios_release.zip) but you may want to try the nightly to try new features.

OF for iOS includes a variety of useful examples in the `/path/to/OF/examples/ios/` directory. As with everything we've seen so far, the examples are a great reference to get up and running quickly.

{{< alert context="info" icon="✌️" >}}
**iOS and Xcode Versions**

The version of iOS on the iPhone and the version of Xcode on the Mac have to be compatible in order to build apps for the phone. Xcode will notify you if the versions are incompatible when plugging the iPhone into the Mac and trying a build.

Apple unfortunately does not support backward compatibility, so you may need to update iOS or Xcode (or both) and there is no way around this.
{{< /alert >}}

### Developer Setup

Setting up Xcode and the device for development takes a few steps. The best way to go through them is to try building an app and resolving issues as they arise.

1. Load the `emptyExample` project in Xcode.
1. Log into your Apple ID to set up a development team.
{{< image src="ios-add-account.png" alt="Xcode Add Account" width="600px" >}}
1. You will probably get a "Failed to register bundle identifier" error. This is because the app needs a unique ID (aka "Bundle Identifier") in the system.
{{< image src="ios-bundle-id-bad.png" alt="Xcode Default Bundle Identifier" width="600px" >}}
1. Common practice is to build an identifier using [reverse domain name notation](https://en.wikipedia.org/wiki/Reverse_domain_name_notation), for example `net.betamovement.seeingmachines.emptyExample`. After changing the value, click the "Try Again" button below.
{{< image src="ios-bundle-id-ok.png" alt="Xcode Unique Bundle Identifier" width="600px" >}}
1. If the status shows the error "Failed to create provisioning profile", it probably means the iOS version on the device is not compatible with this version of Xcode. You can [check which iOS and Xcode versions are compatible](https://developer.apple.com/support/xcode/), and update whichever needs updating until it works. Fun times.
{{< image src="ios-unsupported-os.png" alt="Xcode Unsupported OS" width="600px" >}}
1. If the status at the top has the message "Developer Mode disabled" next to the device name, follow the instructions to enable "Developer Mode".
{{< image src="ios-dev-mode-bad.png" alt="Xcode Developer Mode Disabled" width="600px" >}}
1. On the phone, navigate to the "Privacy & Security" section of the "Settings" app, and enable "Developer Mode" at the bottom. You may need to restart your phone.
{{< image src="ios-dev-mode-phone.jpg" alt="iOS Developer Mode" width="300px" >}}
1. At this point, there should be no more errors and you should be able to build the app in Xcode to run on your device.
{{< image src="ios-dev-mode-ok.png" alt="Xcode Ready" width="600px" >}}

Haha, jk.

The device will now complain about attempting to run an app from an "Untrusted Developer".

1. On the phone, navigate to the "General" section of the "Settings" app, and open the "VPN & Device Management" section.
1. Select the Apple Development account associated with your ID, then tap the "Trust..." button on the next page.
{{< image src="ios-untrusted-combo.jpg" alt="iOS Trust Developer" width="600px" >}}

Try building in Xcode one more time, and it should finally run on the device.

### OF on iOS

OF for iOS is a set of extra files packaged as the `ofxiOS` addon. This addon does not need to be added using the Project Generator, it is automatically included for all iOS projects.

{{< alert context="info" icon="✌️" >}}
**C++ on iOS**

iOS apps were originally written in a language called Objective-C. Over the past several years, Apple has been pushing a modern language called Swift, but Objective-C is still widely supported and is often what runs under the hood of some frameworks.

Objective-C, like C++, is a superset of C and this makes it very easy to compile C++ for iOS. In fact, we can write C, Objective-C, and C++ code in the same classes and the compiler will usually be able to interpret the code correctly.

OF for iOS leverages this compiler to allow writing C++ code on top of an Objective-C layer used for window management and communication with the OS.
{{< /alert >}}

One of the first things we may notice when creating a new iOS OF app is that the file extensions for `main` and `ofApp` have changed from `cpp` to `mm`. `mm` is a special extension that tells the compiler that the code in the file is a mix of both C++ and Objective-C, or that the classes in the file interface with Objective-C objects.

If the classes we create stay in the OF ecosystem (i.e. do not communicate with any iOS frameworks), they can usually keep their `cpp` extension.

`main.mm` also uses a different object to set up the window, and includes many more options.

```cpp
// main.mm
#include "ofApp.h"

int main()
{
  ofiOSWindowSettings settings;
  settings.enableRetina = true; // enables retina resolution if the device supports it.
  settings.enableDepth = false; // enables depth buffer for 3d drawing.
  settings.enableAntiAliasing = false; // enables anti-aliasing which smooths out graphics on the screen.
  settings.numOfAntiAliasingSamples = 0; // number of samples used for anti-aliasing.
  settings.enableHardwareOrientation = false; // enables native view orientation.
  settings.enableHardwareOrientationAnimation = false; // enables native orientation changes to be animated.
  settings.glesVersion = OFXIOS_RENDERER_ES2; // type of renderer to use, ES1, ES2, ES3
  settings.windowControllerType = ofxiOSWindowControllerType::GL_KIT; // Window Controller Type
  settings.colorType = ofxiOSRendererColorFormat::RGBA8888; // color format used default RGBA8888
  settings.depthType = ofxiOSRendererDepthFormat::DEPTH_NONE; // depth format (16/24) if depth enabled
  settings.stencilType = ofxiOSRendererStencilFormat::STENCIL_NONE; // stencil mode
  settings.windowMode = OF_FULLSCREEN;
  settings.enableMultiTouch = false; // enables multitouch support and updates touch.id etc.
  ofCreateWindow(settings);

  return ofRunApp(new ofApp);
}
```

A couple of notable settings to notice here:

* `enableHardwareOrientation` will allow the app to reorient itself as the device is rotated.
* `enableMultiTouch` will enable multitouch input, with fingers being tracked across frames.

The `ofApp` itself will have a few new callback functions available.

```cpp
// ofApp.h
#pragma once

#include "ofxiOS.h"

class ofApp : public ofxiOSApp
{
public:
  void setup();
  void update();
  void draw();
  void exit();

  void touchDown(ofTouchEventArgs & touch);
  void touchMoved(ofTouchEventArgs & touch);
  void touchUp(ofTouchEventArgs & touch);
  void touchDoubleTap(ofTouchEventArgs & touch);
  void touchCancelled(ofTouchEventArgs & touch);

  void lostFocus();
  void gotFocus();
  void gotMemoryWarning();
  void deviceOrientationChanged(int newOrientation);
};
```

* Mouse events are replaced by touch events like `touchDown()`, `touchMoved()`, and `touchDoubleTap()`. These use an [`ofTouchEventArgs`](https://openframeworks.cc/documentation/events/ofTouchEventArgs/#!show_ofTouchEventArgs) object as an argument, which holds touch pointer information like pressure, acceleration, and speed.
* `lostFocus()`  and `gotFocus()` are called when an application is moved to the background or the foreground.
* `deviceOrientationChanged()` is called when the device orientation changes.

### Bouncing Balls

Let's port our bouncing balls example to iOS.

* The `ezBall` class can be used as-is.
* The mouse and keyboard events are converted to use touches instead.
* We will use actual gravity to apply as a force to the balls! We will do this using the `ofxiOSCoreMotion` addon for OF. As this is already part of `ofxiOS`, we just need to include the header and are good to go.

```cpp
// ofApp.h
#pragma once

#include "ofxiOS.h"
#include "ofxiOSCoreMotion.h"

#include "ezBall.h"

class ofApp : public ofxiOSApp
{
public:
  void setup();
  void update();
  void draw();

  void touchDown(ofTouchEventArgs & touch);
  void touchMoved(ofTouchEventArgs & touch);
  //void touchUp(ofTouchEventArgs & touch);
  void touchDoubleTap(ofTouchEventArgs & touch);
  //void touchCancelled(ofTouchEventArgs & touch);

  //void lostFocus();
  //void gotFocus();
  //void gotMemoryWarning();
  //void deviceOrientationChanged(int newOrientation);
  
  void addBall(int x, int y);
  
  ofxiOSCoreMotion coreMotion;
  
  vector<ezBall> balls;
};
```

```cpp
// ofApp.mm
#include "ofApp.h"

void ofApp::setup()
{
  ofBackground(0);
  coreMotion.setupAccelerometer();
}

void ofApp::update()
{
  coreMotion.update();
  glm::vec2 gravity = glm::vec2(coreMotion.getAccelerometerData().x, coreMotion.getAccelerometerData().y * -1);
  for (int i = 0; i < balls.size(); i++)
  {
    balls[i].update(gravity);
  }
}

void ofApp::draw()
{
  for (int i = 0; i < balls.size(); i++)
  {
    balls[i].draw();
  }
}

void ofApp::addBall(int x, int y)
{
  // Add a new ezBall.
  balls.push_back(ezBall());
  // Setup the last added ezBall.
  balls.back().setup(x, y);
}

void ofApp::touchDown(ofTouchEventArgs & touch)
{
  addBall(touch.x, touch.y);
}

void ofApp::touchMoved(ofTouchEventArgs & touch)
{
  addBall(touch.x, touch.y);
}

void ofApp::touchDoubleTap(ofTouchEventArgs & touch)
{
  balls.clear();
}
```

## Android

OF also runs on most Android platforms, including phones, tablets, and microcontrollers.

The latest `0.12.0` release of openFrameworks does not yet work on Android, so we will use the previous release `0.11.2` for the following examples. Resolving this is in the works and will be coming in an update soon.

### Requirements

* [Android Studio](https://developer.android.com/studio/archive) version `3.6.3` to compile apps for Android. This works on Mac, Windows, or Linux platforms. Note that we are using an older release of Android Studio as that is the version compatible with the latest version of OF.
* We will also need to download the Android NDK (Native Development Kit) version `r15c`. Links for each platform are provided on the [OF Setup Guide](https://openframeworks.cc/setup/android-studio/).
* The Android version of openFrameworks. The following code has been tested with [version 0.11.2](https://github.com/openframeworks/openFrameworks/releases/download/0.11.2/of_v0.11.2_android_release.tar.gz).

OF for Android includes a variety of useful examples in the `/path/to/OF/examples/android/` directory. As with everything we've seen so far, the examples are a great reference to get up and running quickly.

The process here is unfortunately also convoluted. Android Studio sometimes has trouble finding required header files, and will not auto-complete correctly or even mark errors in the code even if it compiles the app successfully.

While there are other ways to compile apps for Android, Android Studio is the easiest as it takes care of downloading all required dependencies. If you are having trouble, I would suggest a workflow where code can be written in Xcode or Visual Studio then brought over to Android Studio when it is ready to be tested on a device.

{{< alert context="info" icon="✌️" >}}
**C++ on Android**

Android development is usually done in Java or Kotlin, but there have also always been bindings in C++ available through the [NDK](https://developer.android.com/ndk). This is how OF compiles C++ code and apps for the Android platform.

OF uses the [Gradle](https://gradle.org/) build system to compile apps for Android. Using Gradle decouples building from the IDE, and allows making customizations in the build system without having to go through convoluted menus or being dependent on a specific program. While this has many advantages, it's also the reason why Android Studio will have trouble finding dependencies even if the app is building successfully.
{{< /alert >}}

### OF on Android

OF for Android is a set of extra files packaged as the `ofxAndroid` addon. This addon does not need to be added using the Project Generator, it is automatically included for all Android projects.

Android uses a concept called ["Activities"](https://developer.android.com/guide/components/activities/intro-activities) to organize applications into modules. OF is basically an Android Activity that runs in an app.

`main.cpp` will have an extra section at the bottom to trigger a call to the `main()` function from the OF activity. This is encapsulated in an `#ifdef` compiler directive so that the same file can be used for non-Android apps, and that section will just be ignored.

```cpp
// main.cpp
#include "ofMain.h"
#include "ofApp.h"

int main()
{
  ofSetupOpenGL(1024, 768, OF_WINDOW);
  ofRunApp(new ofApp());
  return 0;
}

#ifdef TARGET_ANDROID
void ofAndroidApplicationInit()
{
  // Application scope init
}

void ofAndroidActivityInit()
{
  // Activity scope init
  main();
}
#endif
```

The `ofApp` itself will have a few new callback functions available.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxAndroid.h"

class ofApp : public ofxAndroidApp
{
public:
  void setup();
  void update();
  void draw();

  void touchDown(int x, int y, int id);
  void touchMoved(int x, int y, int id);
  void touchUp(int x, int y, int id);
  void touchDoubleTap(int x, int y, int id);
  void touchCancelled(int x, int y, int id);
  void swipe(ofxAndroidSwipeDir swipeDir, int id);

  void pause();
  void stop();
  void resume();
  void reloadTextures();

  bool backPressed();
  void okPressed();
  void cancelPressed();
};
```

* Mouse events are replaced by touch events like `touchDown()`, `touchMoved()`, and `touchDoubleTap()`.
* `pause()`  and `resume()` are called when an application is moved to the background or the foreground. `stop()` is called when an application quits.

### Bouncing Balls

Let's port our bouncing balls example to Android.

* The `ezBall` class can be used as-is.
* The mouse and keyboard events are converted to use touches instead.
* We will use actual gravity to apply as a force to the balls! We will do this using the `ofxAccelerometer` addon for OF. As this is already part of `ofxAndroid`, we just need to include the header and are good to go.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxAndroid.h"

#include "ezBall.h"

class ofApp : public ofxAndroidApp
{
public:
  void setup();
  void update();
  void draw();

  void touchDown(int x, int y, int id);
  void touchMoved(int x, int y, int id);
  //void touchUp(int x, int y, int id);
  void touchDoubleTap(int x, int y, int id);
  //void touchCancelled(int x, int y, int id);
  //void swipe(ofxAndroidSwipeDir swipeDir, int id);

  //void pause();
  //void stop();
  //void resume();
  //void reloadTextures();

  bool backPressed();
  //void okPressed();
  //void cancelPressed();
  
  void addBall(int x, int y);

  vector<ezBall> balls;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"
#include "ofxAccelerometer.h"

void ofApp::setup()
{
  ofBackground(0);
  ofxAccelerometer.setup();
}

void ofApp::update()
{
  glm::vec2 gravity = glm::vec2(ofxAccelerometer.getForce().x, ofxAccelerometer.getForce().y * -1);

  for (int i = 0; i < balls.size(); i++)
  {
    balls[i].update(gravity);
  }
}

void ofApp::draw()
{
  for (int i = 0; i < balls.size(); i++)
  {
    balls[i].draw();
  }
}

void ofApp::addBall(int x, int y)
{
  // Add a new ezBall.
  balls.push_back(ezBall());
  // Setup the last added ezBall.
  balls.back().setup(x, y);
}

void ofApp::touchDown(int x, int y, int id)
{
  addBall(x, y);
}

void ofApp::touchMoved(int x, int y, int id)
{
  addBall(x, y);
}

void ofApp::touchDoubleTap(int x, int y, int id)
{
  balls.clear();
}

bool ofApp::backPressed()
{
  return false;
}
```

<figure style="width:600px;height:400px;display:block;margin:0 auto;">
<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/122138826?color=ef0065&title=0&byline=0&portrait=0" style="position:absolute;top:0;left:0;width:100%;height:100%;" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>
<figcaption><i><a href="https://vimeo.com/122138826">Learning Lab: Mars Base</a> from <a href="https://vimeo.com/scatterco">Scatter</a> on <a href="https://vimeo.com">Vimeo</a>.</i></figcaption>
</figure>

<figure style="width:600px;height:400px;display:block;margin:0 auto;">
<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/122138852?color=ef0065&title=0&byline=0&portrait=0" style="position:absolute;top:0;left:0;width:100%;height:100%;" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>
<figcaption><i><a href="https://vimeo.com/122138852">Fitzania</a> from <a href="https://vimeo.com/scatterco">Scatter</a> on <a href="https://vimeo.com">Vimeo</a>.</i></figcaption>
</figure>

## Computer Vision

Things get interesting when we start doing more advanced processing on mobile devices. Let's try to do some computer vision directly on our phones.

### Video

The OF core tries to have parity across all platforms, as such we just need to create an `ofVideoGrabber` as we have been doing so far to get access to the device's camera.

Here is an example of this on Android.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxAndroid.h"

#include "Ball.h"

class ofApp : public ofxAndroidApp
{
public:
  void setup();
  void update();
  void draw();

  //void touchDown(int x, int y, int id);
  //void touchMoved(int x, int y, int id);
  //void touchUp(int x, int y, int id);
  //void touchDoubleTap(int x, int y, int id);
  //void touchCancelled(int x, int y, int id);
  //void swipe(ofxAndroidSwipeDir swipeDir, int id);

  //void pause();
  //void stop();
  //void resume();
  //void reloadTextures();

  bool backPressed();
  //void okPressed();
  //void cancelPressed();

  ofVideoGrabber grabber;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetOrientation(OF_ORIENTATION_90_LEFT);

  grabber.setup(1280, 720);
}

void ofApp::update()
{
  grabber.update();
}

void ofApp::draw()
{
  // Scale using transform matrices.
  // Fill width.
  float scaleRatio = ofGetWidth() / grabber.getWidth();
  float drawX = 0;
  float drawHeight = grabber.getHeight() * scaleRatio;
  float drawY = (ofGetHeight() - drawHeight) / 2.0f;

  ofPushMatrix();
  {
    ofTranslate(drawX, drawY);
    ofScale(scaleRatio);

    ofSetColor(255);
    grabber.draw(0, 0);
  }
  ofPopMatrix();

  ofSetColor(255, 0, 255);
  ofDrawBitmapString(ofToString(ofGetFrameRate(), 2, '0') + "FPS", 100, 10);
}

bool ofApp::backPressed()
{
  return false;
}
```

Note that we can also use the device list to pick between front or back camera! Have a look at the relevant apps in the iOS or Android examples for the code to do so.

### OpenCV

Addons can also be used on mobile platforms, as long as they include all necessary files for the target platforms.

* If an addon consists of sources only, there is a good chance it will run on mobile as the sources will be compiled to the target platform in the build process.
* If an addon includes pre-compiled libraries, these need to include the correct variants to work on mobile platforms.

OpenCV can run almost anywhere, and OF includes the precompiled libraries for both iOS and Android. Let's run a simple thresholding example that uses `ofxCv`. In both cases, we can use the Project Generator to set up our project including all prerequisites.

The following is the version for Android.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxAndroid.h"
#include "ofxCv.h"

class ofApp : public ofxAndroidApp
{
public:
  void setup();
  void update();
  void draw();

  //void touchDown(int x, int y, int id);
  void touchMoved(int x, int y, int id);
  //void touchUp(int x, int y, int id);
  void touchDoubleTap(int x, int y, int id);
  //void touchCancelled(int x, int y, int id);
  //void swipe(ofxAndroidSwipeDir swipeDir, int id);

  //void pause();
  //void stop();
  //void resume();
  //void reloadTextures();

  bool backPressed();
  //void okPressed();
  //void cancelPressed();

  ofVideoGrabber grabber;
  ofImage thresholdImg;
  ofxCv::ContourFinder contourFinder;

  ofParameter<int> thresholdVal;
  ofParameter<bool> drawThreshold;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetOrientation(OF_ORIENTATION_90_LEFT);

  grabber.setPixelFormat(OF_PIXELS_GRAY);
  grabber.setup(1280, 720);

  thresholdImg.allocate(1280, 720, OF_IMAGE_GRAYSCALE);

  thresholdVal.set("Threshold Val", 127, 0, 255);
  drawThreshold.set("Draw Threshold", true);
}

void ofApp::update()
{
  grabber.update();
  if (grabber.isFrameNew())
  {
    ofxCv::threshold(grabber, thresholdImg, thresholdVal);
    contourFinder.findContours(thresholdImg);
  }
}

void ofApp::draw()
{
  // Scale using transform matrices.
  // Fill width.
  float scaleRatio = ofGetWidth() / grabber.getWidth();
  float drawX = 0;
  float drawHeight = grabber.getHeight() * scaleRatio;
  float drawY = (ofGetHeight() - drawHeight) / 2.0f;

  ofPushMatrix();
  {
    ofTranslate(drawX, drawY);
    ofScale(scaleRatio);

    ofSetColor(255);
    if (drawThreshold)
    {
      // Only update the image if we need to draw it.
      thresholdImg.update();
      thresholdImg.draw(0, 0);
    } else
    {
      grabber.draw(0, 0);
    }

    ofSetColor(0, 255, 0);
    contourFinder.draw();
  }
  ofPopMatrix();

  ofSetColor(255, 0, 255);
  ofDrawBitmapString(ofToString(ofGetFrameRate(), 2, '0') + "FPS", 100, 10);
}

void ofApp::touchMoved(int x, int y, int id)
{
  thresholdVal = ofMap(x, 0, ofGetWidth(), 0, 255);
}

void ofApp::touchDoubleTap(int x, int y, int id)
{
  drawThreshold = !drawThreshold;
}

bool ofApp::backPressed()
{
  return false;
}
```

<figure style="width:600px;height:400px;display:block;margin:0 auto;">
<video src="android-cv.mp4" controls width="100%"></video>
<figcaption><i>OpenCV Thresholding</i></figcaption>
</figure>

And the following is the version for iOS.

```cpp
// ofApp.h
#pragma once

#include "ofxiOS.h"
#include "ofxCv.h"

class ofApp : public ofxiOSApp
{
public:
  void setup();
  void update();
  void draw();
  //void exit();

  //void touchDown(ofTouchEventArgs & touch);
  void touchMoved(ofTouchEventArgs & touch);
  //void touchUp(ofTouchEventArgs & touch);
  void touchDoubleTap(ofTouchEventArgs & touch);
  //void touchCancelled(ofTouchEventArgs & touch);

  //void lostFocus();
  //void gotFocus();
  //void gotMemoryWarning();
  //void deviceOrientationChanged(int newOrientation);
  
  ofVideoGrabber grabber;

  ofImage thresholdImg;
  ofxCv::ContourFinder contourFinder;
  
  ofFbo renderFbo;
  
  ofParameter<int> thresholdVal;
  ofParameter<bool> drawThreshold;
};
```

```cpp
// ofApp.mm
#include "ofApp.h"

void ofApp::setup()
{
  grabber.setup(640, 480);
  
  thresholdImg.allocate(640, 480, OF_IMAGE_GRAYSCALE);
  
  renderFbo.allocate(640, 480);
  
  thresholdVal.set("Threshold Val", 127, 0, 255);
  drawThreshold.set("Draw Threshold", true);
}

void ofApp::update()
{
  grabber.update();
  if (grabber.isFrameNew())
  {
    ofxCv::convertColor(grabber, thresholdImg, CV_RGB2GRAY);
    ofxCv::threshold(thresholdImg, thresholdVal);
    contourFinder.findContours(thresholdImg);
  }
}

void ofApp::draw()
{
  renderFbo.begin();
  {
    ofSetColor(255);
    if (drawThreshold)
    {
      // Only update the image if we need to draw it.
      thresholdImg.update();
      thresholdImg.draw(0, 0);
    } else
    {
      grabber.draw(0, 0);
    }
    
    ofSetColor(0, 255, 0);
    contourFinder.draw();
  }
  renderFbo.end();
  
  ofRectangle drawBounds = ofRectangle(0, 0, renderFbo.getWidth(), renderFbo.getHeight());
  drawBounds.scaleTo(ofGetCurrentViewport(), OF_SCALEMODE_FILL);
  
  ofSetColor(255);
  renderFbo.draw(drawBounds);
  
  ofSetColor(255, 0, 255);
  ofDrawBitmapString(ofToString(ofGetFrameRate(), 2, '0') + "FPS", 100, 10);
}

void ofApp::touchMoved(ofTouchEventArgs & touch)
{
  thresholdVal = ofMap(touch.x, 0, ofGetWidth(), 0, 255);
}

void ofApp::touchDoubleTap(ofTouchEventArgs & touch)
{
  drawThreshold = !drawThreshold;
}
```
