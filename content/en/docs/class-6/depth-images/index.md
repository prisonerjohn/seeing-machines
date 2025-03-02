---
title: "Depth Images"
description: ""
lead: ""
date: 2022-10-23T14:19:40-04:00
lastmod: 2022-10-23T14:19:40-04:00
draft: false
images: []
menu:
  docs:
    parent: "class-6"
    identifier: "depth-images"
weight: 620
toc: true
---

## Depth Grabbers

While depth sensors come in a variety of shapes, sizes, and technologies, the method to interface with them is usually pretty similar. Just like color cameras, depth cameras deliver images at the requested framerate. These images come in as arrays of pixels, which can then be manipulated and uploaded to a texture for rendering.

As these are specialty devices, we cannot use an `ofVideoGrabber` to retrieve data from them. We will need to use a special "grabber" tailored to each device and its SDK.

{{< alert context="info" icon="✌️" >}}
**How come `ofVideoGrabber` works for all webcams?**

[USB Video Class](https://en.m.wikipedia.org/wiki/USB_video_device_class), or UVC, is a standard describing formats by which a device can stream video frames. Most USB webcams follow this standard and that is why we can use a common interface to access their streams.

`ofVideoGrabber` is therefore able to work with any USB video device that complies with the UVC standard. It is actually a wrapper with specific implementations based on the type of system we are on, for example [AVFoundation](https://developer.apple.com/av-foundation/) on macOS, [GStreamer](https://gstreamer.freedesktop.org/) on Linux, and [Windows Media Foundation](https://docs.microsoft.com/en-us/windows/win32/medfound/microsoft-media-foundation-sdk) on Windows.
{{< /alert >}}

We fortunately will not have to implement custom depth grabbers ourselves. In the same way that `ofxCv` acts as a bridge between OpenCV and openFrameworks, there are many [OF addons](http://ofxaddons.com/categories) we can use that manage the interface between the sensor SDK and openFrameworks.

### Intel RealSense

[`ofxRealSense2`](https://gitlab.com/prisonerjohn/ofxrealsense2/) is a good choice for the Intel RealSense, as it gives us both pixel and texture access, as well as control over many of the SDK's filtering options.

{{< alert context="danger" icon="⚠️" >}}
Unfortunately, there seems to be an [incompatibility with the Intel RealSense and macOS Monterey](https://support.intelrealsense.com/hc/en-us/community/posts/4548413451539-Activating-the-realsense-D435-Depth-Camera-in-MacOs) (and later) related to permissions. You may be unable to run the following code if you are using this operating system.

I am actively looking for solutions and will let the class know once I have more information. This [blog post](https://lightbuzz.com/realsense-macos/) has potential solutions.
{{< /alert >}}

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxRealSense2.h"

class ofApp : public ofBaseApp
{
public:
    void setup();
    void update();
    void draw();
    ofxRealSense2::Context rsContext;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  // Default RS resolution.
  ofSetWindowShape(640, 360);

  // true parameter starts the camera automatically.
  rsContext.setup(true);
}

void ofApp::update()
{
  rsContext.update();
}

void ofApp::draw()
{
  // Try to get a pointer to a device.
  std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    // Draw the depth texture.
    rsDevice->getDepthTex().draw(0, 0);
  }
}
```

A depth image will usually be single-channel, and therefore rendered in grayscale. Each gray value represents the distance of that pixel to the camera. The convention is usually brighter for nearer objects, but this is just representative.

{{< image src="rs-depth.png" alt="RealSense Depth Image" caption="*RealSense Depth Image*" width="600px" >}}

We can read the actual depth value using the SDK function `ofxRealSense2::Device.getDistance(x, y)`. We will read the value under the mouse, and display it using [`ofDrawBitmapStringHighlight()`](https://openframeworks.cc/documentation/graphics/ofGraphics/#!show_ofDrawBitmapString) for debugging.

```cpp
// ofApp.cpp
include "ofApp.h"

void ofApp::setup()
{
  // Default RS resolution.
  ofSetWindowShape(640, 360);

  rsContext.setup(true);
}

void ofApp::update()
{
  rsContext.update();
}

void ofApp::draw()
{
  std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    rsDevice->getDepthTex().draw(0, 0);

    float distAtMouse = rsDevice->getDistance(ofGetMouseX(), ofGetMouseY());
    ofDrawBitmapStringHighlight(ofToString(distAtMouse, 3), ofGetMouseX(), ofGetMouseY());
  }
}
```

The value returned by the SDK is probably read from the depth texture. Let's try to read it directly from the pixels array and see if the values match.

Note that there are usually two available depth readings:

* The raw depth buffer contains the actual depth measurement.
  * The value can be read directly from the pixel value.
  * The value is metric, usually in millimeters (1mm = 0.001m).
  * The pixel format is `unsigned short`, with range `0` to `65535` (16-bit).
* The depth buffer contains a scaled representation of the pixel data.
  * This is just for us to make sure everything is working, as the raw depth image is usually very dark or very bright.
  * We should not use this for any depth readings.
  * The pixel format is `unsigned char`, with range `0` to `255` (8-bit). This is the same as most RGB images.
  * This is sometimes called the *scaled* or *mapped* image.

We will therefore read our value from the raw depth texture.

```cpp
// ofApp.cpp
include "ofApp.h"

void ofApp::setup()
{
  // Default RS resolution.
  ofSetWindowShape(640, 360);

  rsContext.setup(true);
}

void ofApp::update()
{
  rsContext.update();
}

void ofApp::draw()
{
  std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    rsDevice->getDepthTex().draw(0, 0);

    // Get the point distance using the SDK function.
    float distAtMouse = rsDevice->getDistance(ofGetMouseX(), ofGetMouseY());
    ofDrawBitmapStringHighlight(ofToString(distAtMouse, 3), ofGetMouseX(), ofGetMouseY() - 10);

    // Get the point depth using the texture directly.
    ofShortPixels rawDepthPix = rsDevice->getRawDepthPix();
    int depthAtMouse = rawDepthPix.getColor(ofGetMouseX(), ofGetMouseY()).r;
    ofDrawBitmapStringHighlight(ofToString(depthAtMouse), ofGetMouseX() + 16, ofGetMouseY() + 10);
  }
}
```

Note that we are getting an `ofColor` from the depth pixels, and reading the red channel with `ofColor.r` to get the depth value. We could use any of the red, green, blue channels here; as our data is in a single grayscale channel, all the colors represent the same value.

The Intel RealSense raw image tends to be very noisy, and needs some filtering to clean it up and make it usable. The SDK includes many options for filtering and these are available in the addon. To use them with `ofxGui`, we just need to add the `ofxRealSense2::Device::params` object to the `ofxPanel`.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"
#include "ofxRealSense2.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  ofxRealSense2::Context rsContext;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  // Default RS resolution.
  ofSetWindowShape(640, 360);

  guiPanel.setup("Depth", "settings.json");

  rsContext.setup(true);

  std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    guiPanel.add(rsDevice->params);
  }
}

void ofApp::update()
{
  rsContext.update();
}

void ofApp::draw()
{
  std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    rsDevice->getDepthTex().draw(0, 0);

    float distAtMouse = rsDevice->getDistance(ofGetMouseX(), ofGetMouseY());
    ofDrawBitmapStringHighlight(ofToString(distAtMouse, 3), ofGetMouseX(), ofGetMouseY());
  }

  guiPanel.draw();
}
```

### Microsoft Kinect

[`ofxKinect`](https://openframeworks.cc/documentation/ofxKinect/) is the best choice for the original Microsoft Kinect, as it ships with OF and gives us all the data we need.

Notice that the code looks almost similar to what we just did for the RealSense!

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxKinect.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  ofxKinect kinect;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  kinect.init();
  kinect.open();
}

void ofApp::update()
{
  kinect.update();
}

void ofApp::draw()
{
  if (kinect.isFrameNew())
  {
    kinect.getDepthTexture().draw(0, 0);

    // Get the point distance using the SDK function.
    float distAtMouse = kinect.getDistanceAt(ofGetMouseX(), ofGetMouseY());
    ofDrawBitmapStringHighlight(ofToString(distAtMouse, 3), ofGetMouseX(), ofGetMouseY() - 10);

    // Get the point depth using the texture directly.
    ofShortPixels rawDepthPix = kinect.getRawDepthPixels();
    int depthAtMouse = rawDepthPix.getColor(ofGetMouseX(), ofGetMouseY()).r;
    ofDrawBitmapStringHighlight(ofToString(depthAtMouse), ofGetMouseX() + 16, ofGetMouseY() + 10);
  }
}
```

### Microsoft Kinect V2

[`ofxKinectForWindows2`](https://github.com/elliotwoods/ofxKinectForWindows2) is a good choice for the Kinect V2. It works with the [Microsoft Kinect for Windows 2.0 SDK](https://www.microsoft.com/en-us/download/details.aspx?id=44561), which means it supports all Kinect features (including body tracking). However, note that this only works on Windows!

`ofxKinectForWindows2` does not include a function to get distance from a coordinate, so we will need to sample the depth texture directly.

```cpp
// ofApp.h
#include "ofMain.h"

#include "ofxKinectForWindows2.h"

class ofApp : public ofBaseApp 
{
public:
  void setup();
  void update();
  void draw();

  ofxKFW2::Device kinect;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  kinect.open();
  kinect.initDepthSource();
  kinect.initColorSource();
}

void ofApp::update()
{
  kinect.update();
}

void ofApp::draw()
{
  if (kinect.isFrameNew())
  {
    std::shared_ptr<ofxKFW2::Source::Depth> depthSource = kinect.getDepthSource();

    // Clamp the mouse coordinate to ensure it stays within the data bounds.
    int readX = ofClamp(ofGetMouseX(), 0, depthSource->getWidth() - 1);
    int readY = ofClamp(ofGetMouseY(), 0, depthSource->getHeight() - 1);

    // Get the point depth using the texture directly.
    ofShortPixels rawDepthPix = depthSource->getPixels();
    int depthAtMouse = rawDepthPix.getColor(readX, readY).r;
    ofDrawBitmapStringHighlight(ofToString(depthAtMouse), ofGetMouseX(), ofGetMouseY());
  }
}
```

Alternatively, [`ofxKinectV2`](https://github.com/ofTheo/ofxKinectV2) is a cross-platform solution that works similarly to `ofxKinect`. The code to sample the distance under the mouse is very similar.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxKinectV2.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  ofxKinectV2 kinect;
  ofTexture depthTex;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(512, 424);

  // Use a settings object to configure the device.
  ofxKinectV2::Settings settings;
  settings.enableRGB = false;
  settings.enableDepth = true;

  kinect.open(0, settings);
}

void ofApp::update()
{
  kinect.update();

  // Only load the data if there is a new frame to process.
  if (kinect.isFrameNew())
  {
    depthTex.loadData(kinect.getDepthPixels());
  }
}

void ofApp::draw()
{
  depthTex.draw(0, 0);

  // Get the point distance using the SDK function (in meters).
  float distAtMouse = kinect.getDistanceAt(ofGetMouseX(), ofGetMouseY());
  ofDrawBitmapStringHighlight(ofToString(distAtMouse, 3), ofGetMouseX(), ofGetMouseY() - 10);

  // Get the point depth using the texture directly (in millimeters).
  const ofFloatPixels& rawDepthPix = kinect.getRawDepthPixels();
  int depthAtMouse = rawDepthPix.getColor(ofGetMouseX(), ofGetMouseY()).r;
  ofDrawBitmapStringHighlight(ofToString(depthAtMouse), ofGetMouseX() + 16, ofGetMouseY() + 10);
}
```

Some notes to consider:

* The device is configured using a settings object of type `ofxKinectV2::Settings`. This is a common pattern in openFrameworks that we will encounter again.
* `ofxKinectV2` does not provide textures for the data, only pixels. We need to use our own texture and load it with pixel data in `update()`. We use `isFrameNew()` to check if there is new data to upload on each frame.
* The SDK function `getDistanceAt()` returns the distance in meters but the raw pixel data returns the data in millimeters. The depth data is also using `float` pixels instead of the more common `short`.

{{< alert context="info" icon="✌️" >}}
**The `const` qualifier**

[`const`](https://en.cppreference.com/w/cpp/language/cv) is a qualifier that can be used on a variable to indicate that it will not change; that it will remain *constant*.

In the example above, we are creating a temporary variable for the pixel data from `getRawDepthPixels()`. We do not want to make a copy of this data, so we use the `&` when declaring the variable to indicate it will be a reference. This data should not be modified by the programmer because it comes directly from the device. `getRawDepthPixels()` indicates this in its return type; it requires any reference to be constant.
{{< /alert >}}

## Depth Threshold

Depth pixels are very useful for thresholding images. This tends to be much more precise than using brightness or color (as we have been doing previously) since we eliminate any issues with change in lighting or with similarities between background and foreground colors. We can set a depth range that valid pixels belong to and discard anything that's nearer or farther than this range.

Let's first attempt to do this manually by iterating through the pixel array.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"
#include "ofxRealSense2.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  ofxRealSense2::Context rsContext;

  ofImage thresholdImg;

  ofParameter<int> nearThreshold;
  ofParameter<int> farThreshold;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  // Default RS resolution.
  ofSetWindowShape(640, 720);

  // Start the sensor.
  rsContext.setup(true);

  // Allocate the image.
  thresholdImg.allocate(640, 360, OF_IMAGE_GRAYSCALE);

  // Setup the parameters.
  nearThreshold.set("Near Threshold", 10, 0, 4000);
  farThreshold.set("Far Threshold", 1000, 0, 4000);

  // Setup the gui.
  guiPanel.setup("Depth Threshold", "settings.json");
  guiPanel.add(nearThreshold);
  guiPanel.add(farThreshold);
}

void ofApp::update()
{
  rsContext.update();
}

void ofApp::draw()
{
  std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    rsDevice->getDepthTex().draw(0, 0);

    // Get the point distance using the SDK function.
    float distAtMouse = rsDevice->getDistance(ofGetMouseX(), ofGetMouseY());
    ofDrawBitmapStringHighlight(ofToString(distAtMouse, 3), ofGetMouseX(), ofGetMouseY());

    // Threshold the depth.
    ofShortPixels rawDepthPix = rsDevice->getRawDepthPix();
    ofPixels& thresholdPix = thresholdImg.getPixels();
    for (int y = 0; y < rawDepthPix.getHeight(); y++)
    {
      for (int x = 0; x < rawDepthPix.getWidth(); x++)
      {
        int depth = rawDepthPix.getColor(x, y).r;
        if (nearThreshold < depth && depth < farThreshold)
        {
          thresholdPix.setColor(x, y, ofColor(255));
        }
        else
        {
          thresholdPix.setColor(x, y, ofColor(0));
        }
      }
    }

    // Upload pixels to texture.
    thresholdImg.update();

    // Draw the result image.
    thresholdImg.draw(0, 360);
  }

  // Draw the gui.
  guiPanel.draw();
}
```

{{< image src="rs-threshold.png" alt="RealSense Threshold" caption="*RealSense Threshold*" width="600px" >}}

We can also achieve the same effect using OpenCV with two consecutive calls to [`cv::threshold()`](https://docs.opencv.org/2.4/modules/imgproc/doc/miscellaneous_transformations.html?highlight=threshold#threshold).

* The first will be for the near value, and will keep everything greater than the threshold value.
* The second will be for the far value, and will be inverted so that we keep everything smaller than the threshold value.
* We then combine the result of both operations using [`cv::bitwise_and()`](https://docs.opencv.org/2.4/modules/core/doc/operations_on_arrays.html?highlight=bitwise_and#bitwise-and), which just adds both textures together.

Unfortunately, OpenCV does not work with `unsigned short` images, so we cannot use our array of depth pixels directly. We first need to convert it either to an array of `unsigned char` or `float`.

* If we go with `unsigned char`, we will lose precision because we will need to pack `65536` possible values into `256`. We should therefore use `float` and `ofFloatPixels`.
* In both cases, if we let OF do the conversion automatically, it will rescale the values to fit into the new range. So `[0, 65535]` becomes `[0, 255]` or `[0.0, 1.0]`. We need to remain aware of this as it will change the range of our near/far parameters.

Here is a second thresholding attempt using OpenCV.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxCv.h"
#include "ofxGui.h"
#include "ofxRealSense2.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  ofxRealSense2::Context rsContext;

  ofImage thresholdImg;

  ofParameter<float> nearThreshold;
  ofParameter<float> farThreshold;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  // Default RS resolution.
  ofSetWindowShape(640, 720);

  // Start the sensor.
  rsContext.setup(true);

  // Setup the parameters.
  nearThreshold.set("Near Threshold", 0.01f, 0.0f, 0.1f);
  farThreshold.set("Far Threshold", 0.02f, 0.0f, 0.1f);

  // Setup the gui.
  guiPanel.setup("Depth Threshold", "settings.json");
  guiPanel.add(nearThreshold);
  guiPanel.add(farThreshold);
}

void ofApp::update()
{
  rsContext.update();
}

void ofApp::draw()
{
  std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    rsDevice->getDepthTex().draw(0, 0);

    // Get the point distance using the SDK function.
    float distAtMouse = rsDevice->getDistance(ofGetMouseX(), ofGetMouseY());
    ofDrawBitmapStringHighlight(ofToString(distAtMouse, 3), ofGetMouseX(), ofGetMouseY());

    // Threshold the depth.
    ofFloatPixels rawDepthPix = rsDevice->getRawDepthPix();
    ofFloatPixels thresholdNear, thresholdFar, thresholdResult;
    ofxCv::threshold(rawDepthPix, thresholdNear, nearThreshold);
    ofxCv::threshold(rawDepthPix, thresholdFar, farThreshold, true);
    ofxCv::bitwise_and(thresholdNear, thresholdFar, thresholdResult);

    // Upload pixels to image.
    thresholdImg.setFromPixels(thresholdResult);

    // Draw the result image.
    thresholdImg.draw(0, 360);
  }

  // Draw the gui.
  guiPanel.draw();
}
```

And here is that same example using a Kinect and OpenCV.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxCv.h"
#include "ofxGui.h"
#include "ofxKinect.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  ofxKinect kinect;

  ofImage thresholdImg;

  ofParameter<float> nearThreshold;
  ofParameter<float> farThreshold;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(1280, 480);

  // Start the depth sensor.
  kinect.setRegistration(true);
  kinect.init();
  kinect.open();

  // Setup the parameters.
  nearThreshold.set("Near Threshold", 0.01f, 0.0f, 0.1f);
  farThreshold.set("Far Threshold", 0.02f, 0.0f, 0.1f);

  // Setup the gui.
  guiPanel.setup("Depth Threshold", "settings.json");
  guiPanel.add(nearThreshold);
  guiPanel.add(farThreshold);
}

void ofApp::update()
{
  kinect.update();
}

void ofApp::draw()
{
  if (kinect.isFrameNew())
  {
    // Get the point distance using the SDK function.
    float distAtMouse = kinect.getDistanceAt(ofGetMouseX(), ofGetMouseY());
    ofDrawBitmapStringHighlight(ofToString(distAtMouse, 3), ofGetMouseX(), ofGetMouseY());

    // Threshold the depth.
    ofFloatPixels rawDepthPix = kinect.getRawDepthPixels();
    ofFloatPixels thresholdNear, thresholdFar, thresholdResult;
    ofxCv::threshold(rawDepthPix, thresholdNear, nearThreshold);
    ofxCv::threshold(rawDepthPix, thresholdFar, farThreshold, true);
    ofxCv::bitwise_and(thresholdNear, thresholdFar, thresholdResult);

    // Upload pixels to image.
    thresholdImg.setFromPixels(thresholdResult);
  }

  // Draw the source image.
  kinect.getDepthTexture().draw(0, 0);

  // Draw the result image.
  thresholdImg.draw(640, 0);

  // Draw the gui.
  guiPanel.draw();
}
```
