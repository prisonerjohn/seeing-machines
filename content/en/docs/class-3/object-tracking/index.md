---
title: "Object Tracking"
description: "Object Tracking"
lead: ""
date: 2022-10-01T20:48:41-04:00
lastmod: 2022-10-01T20:48:41-04:00
draft: false
images: []
menu:
  docs:
    parent: "class-3"
    identifier: "object-tracking"
weight: 420
toc: true
---

Let's build an app together that tracks an object in the frame over time.

We are going to track a red playing card and can assume that the card will be the "most red" element in the frame.

{{< image src="red-card.jpg" alt="A Red Card" width="600px" >}}

## Contour Finding

In order to do this, we will use a *Contour Finding* algorithm.

Contour finding consists of identifying regions of images matching a particular pattern. These patterns can be defined by color, size, shape, speed, etc. Contour finding can be used to follow objects or people in an image.

<figure style="width:600px;height:400px;display:block;margin:0 auto;" markdown="1">
<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/141447523" style="position:absolute;top:0;left:0;width:100%;height:100%;" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>
<figcaption><i><a href="https://vimeo.com/141447523">BloodBank - Video game</a> from <a href="https://vimeo.com/vincentdevevey">Vincent de Vevey</a> on <a href="https://vimeo.com">Vimeo</a>.</i></figcaption>
</figure>

If we break it down, this usually consist of many steps:

1. Convert the pixel data to an appropriate color space.
1. Threshold the image based on a target color and offset. As we saw previously, video pixel colors vary over frames, so we need to use an offset to look for a color range.
1. Run the OpenCV contour finding operation on the image.
1. Filter the array of contours and only keep the ones that match the requested parameters.

This is where `ofxCv` comes in very handy, as we can use the `ofxCv::ContourFinder` class to handle a big part of the work.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxCv.h"
#include "ofxGui.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  ofVideoGrabber grabber;

  ofxCv::ContourFinder contourFinder;

  ofParameter<ofColor> colorTarget;
  ofParameter<int> colorOffset;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  // Setup the grabber.
  grabber.setup(640, 480);

  // Setup the contour finder and parameters.
  contourFinder.setUseTargetColor(true);

  colorTarget.set("Color Target", ofColor(255, 0, 0));
  colorOffset.set("Color Offset", 10, 0, 255);

  // Setup the gui.
  guiPanel.setup("Color Tracker", "settings.json");
  guiPanel.add(colorTarget);
  guiPanel.add(colorOffset);
}

void ofApp::update()
{
  grabber.update();
  if (grabber.isFrameNew())
  {
    // Update parameters.
    contourFinder.setTargetColor(colorTarget);
    contourFinder.setThreshold(colorOffset);

    // Find contours.
    contourFinder.findContours(grabber);
  }
}

void ofApp::draw()
{
  ofSetColor(255);

  // Draw the grabber image.
  grabber.draw(0, 0, ofGetWidth(), ofGetHeight());

  // Draw the found contours.
  contourFinder.draw();

  // Draw the gui.
  guiPanel.draw();
}
```

## Color Space

While this application technically works, it is hard to get the settings right. Let's modify it to make it easier to use.

We often get better results when comparing colors in HSV rather than RGB space.

* RGB defines how much red, green, and blue is in an image.
  * A small change in values might result in a greater difference seen, and vice-versa.
  * When working near the grayscale range (white to black), it is hard to evaluate how much red, green, and blue is actually in the pixel.
* HSV defines colors as levels of hue, saturation, and brightness.
  * This is closer to how humans perceive color, and differentiate objects they see in space.
  * It is easier to isolate parameters. We will often just care about the brightness of an image, particularly in the grayscale range.
  * It is a more logical set of parameters for many image analysis algorithms.

We can tell `ofxCv::ContourFinder` to use HSV by passing a second parameter to the `setTargetColor()` method:

```cpp
contourFinder.setTargetColor(colorTarget, ofxCv::TRACK_COLOR_HSV);
```

## Mouse Selector

We can use the mouse to interactively select the color under the cursor. This will remove any guesswork from setting the target color accurately.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxCv.h"
#include "ofxGui.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  void mousePressed(int x, int y, int button);

  ofVideoGrabber grabber;
  ofImage processImg;

  ofxCv::ContourFinder contourFinder;

  ofParameter<ofColor> colorTarget;
  ofParameter<int> colorOffset;
  
  ofColor colorUnderMouse;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  // Setup the grabber.
  grabber.setup(640, 480);

  // Setup the contour finder and parameters.
  contourFinder.setUseTargetColor(true);

  colorTarget.set("Color Target", ofColor(255, 0, 0));
  colorOffset.set("Color Offset", 10, 0, 255);

  // Setup the gui.
  guiPanel.setup("Color Tracker", "settings.json");
  guiPanel.add(colorTarget);
  guiPanel.add(colorOffset);
}

void ofApp::update()
{
  grabber.update();
  if (grabber.isFrameNew())
  {
    processImg.setFromPixels(grabber.getPixels());

    // Save the color of the pixel under the mouse.
    colorUnderMouse = processImg.getColor(ofGetMouseX(), ofGetMouseY());

    // Update parameters.
    contourFinder.setTargetColor(colorTarget, ofxCv::TRACK_COLOR_HSV);
    contourFinder.setThreshold(colorOffset);

    // Find contours.
    contourFinder.findContours(processImg);
  }
}

void ofApp::draw()
{
  ofSetColor(255);

  // Draw the grabber image.
  grabber.draw(0, 0, ofGetWidth(), ofGetHeight());

  // Draw the found contours.
  contourFinder.draw();
  
  // Draw the color under the mouse.
  ofPushStyle();
  ofSetColor(colorUnderMouse);
  ofDrawRectangle(ofGetMouseX() - 25, ofGetMouseY() - 25, 50, 50);
  ofNoFill();
  ofSetColor(colorUnderMouse.getInverted());
  ofDrawRectangle(ofGetMouseX() - 25, ofGetMouseY() - 25, 50, 50);
  ofPopStyle();

  // Draw the gui.
  guiPanel.draw();
}

void ofApp::mousePressed(int x, int y, int button)
{
  if (!guiPanel.getShape().inside(x, y))
  {
    // Track the color under the mouse.
    colorTarget = colorUnderMouse;
  }
}
```

## Area Range

The contour finder seems to be working too well. We are getting many blobs, and most are too big or too small to consider.

We can limit the range of blob sizes to look for, and only match the ones within this range.

`ofxCv::ContourFinder` gives us a few different options for this. In all cases, we are looking at the area of the blob bounding box. The difference is simply in how this gets calculated.

* `setMinArea()` / `setMaxArea()` set the range in pixels. The max range depends on the size of the image as a larger image will have more pixel area.
  * e.g. If a blob's dimensions are 20px width by 10px height, the area is `w x h` = 200px<sup>2</sup>.
* `setMinAreaRadius()` / `setMaxAreaRadius()` set the area using the blob radius. This can be useful if the blobs we are looking for are circular in shape.
* `setMinAreaNorm()` / `setMaxAreaNorm()` set the area using *normalized* coordinates. This means the range is always between `0.0` and `1.0`, no matter what the image size is.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxCv.h"
#include "ofxGui.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  void mousePressed(int x, int y, int button);

  ofVideoGrabber grabber;
  ofImage processImg;

  ofxCv::ContourFinder contourFinder;

  ofParameter<ofColor> colorTarget;
  ofParameter<int> colorOffset;
  
  ofColor colorUnderMouse;

  ofParameter<float> minArea;
  ofParameter<float> maxArea;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  // Setup the grabber.
  grabber.setup(640, 480);

  // Setup the contour finder and parameters.
  contourFinder.setUseTargetColor(true);

  colorTarget.set("Color Target", ofColor(255, 0, 0));
  colorOffset.set("Color Offset", 10, 0, 255);
  minArea.set("Min Area", 0.01f, 0, 0.5f);
  maxArea.set("Max Area", 0.05f, 0, 0.5f);

  // Setup the gui.
  guiPanel.setup("Color Tracker", "settings.json");
  guiPanel.add(colorTarget);
  guiPanel.add(colorOffset);
  guiPanel.add(minArea);
  guiPanel.add(maxArea);
}

void ofApp::update()
{
  grabber.update();
  if (grabber.isFrameNew())
  {
    processImg.setFromPixels(grabber.getPixels());

    // Save the color of the pixel under the mouse.
    colorUnderMouse = processImg.getColor(ofGetMouseX(), ofGetMouseY());

    // Update parameters.
    contourFinder.setTargetColor(colorTarget, ofxCv::TRACK_COLOR_HSV);
    contourFinder.setThreshold(colorOffset);
    contourFinder.setMinAreaNorm(minArea);
    contourFinder.setMaxAreaNorm(maxArea);

    // Find contours.
    contourFinder.findContours(processImg);
  }
}

void ofApp::draw()
{
  ofSetColor(255);

  // Draw the grabber image.
  grabber.draw(0, 0, ofGetWidth(), ofGetHeight());
  
  // Draw the found contours.
  contourFinder.draw();

  // Draw the color under the mouse.
  ofPushStyle();
  ofSetColor(colorUnderMouse);
  ofDrawRectangle(ofGetMouseX() - 25, ofGetMouseY() - 25, 50, 50);
  ofNoFill();
  ofSetColor(colorUnderMouse.getInverted());
  ofDrawRectangle(ofGetMouseX() - 25, ofGetMouseY() - 25, 50, 50);
  ofPopStyle();

  // Draw the gui.
  guiPanel.draw();
}

void ofApp::mousePressed(int x, int y, int button)
{
  if (!guiPanel.getShape().inside(x, y))
  {
    // Track the color under the mouse.
    colorTarget = colorUnderMouse;
  }
}
```

<figure style="width:600px;height:400px;display:block;margin:0 auto;" markdown="1">
<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/1859773" style="position:absolute;top:0;left:0;width:100%;height:100%;" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>
<figcaption><i><a href="https://vimeo.com/1859773">Royal Opera House - Audience: &quot;chatting and following&quot;</a> from <a href="https://vimeo.com/randomvids">RANDOM INTERNATIONAL</a> on <a href="https://vimeo.com">Vimeo</a>.</figcaption></i>
</figure>

## Filtering

The pattern on the back of the card is making it hard to get a clean blob. It is getting separated into different parts. Let's try to filter the image before sending it to the contour finder to get better results.

The following operations are called *convolution* operations. A convolution is a process by which a pixel looks at its neighbours values to calculate its own value. The rules to calculate this value are set in a *kernel*.

A *kernel*  has a size and weights.

* The size, also called the *window*, specifies how many neighbours to look at when making the calculation. For example, a `3x3` kernel will consider the 8 direct neighbours and the pixel itself.
* The weights represent how much of each pixel in the kernel to take in when calculating the final value. A *normalized* kernel will have the same weights throughout, while a non-normalized one will have different weight values.
* The window is usually odd and square, also called *box*, (e.g. `3x3` or `5x5`) and the pixel in question is in the middle of it.

```python
                          [ 1  4  6  4 1 ]
[ 1 1 1 ]    [ 1 2 1 ]    [ 4 16 24 16 4 ]
[ 1 1 1 ]    [ 2 4 2 ]    [ 6 24 36 24 6 ]
[ 1 1 1 ]    [ 1 2 1 ]    [ 4 16 24 16 4 ]
                          [ 1  4  6  4 1 ]
```

### Blur

Blurring the image is a good first step to smooth out unwanted details in an image and make it more homogenous.

`ofxCv` offers a few options for blurring:

* [`cv::blur()`](https://docs.opencv.org/master/d4/d86/group__imgproc__filter.html#ga8c45db9afe636703801b0b2e440fce37) uses a normalized kernel.
* [`cv::GaussianBlur()`](https://docs.opencv.org/master/d4/d86/group__imgproc__filter.html#gaabe8c836e97159a9193fb0b11ac52cf1) uses a weighted [Gaussian](https://en.wikipedia.org/wiki/Gaussian_blur) kernel.
* [`cv::medianBlur()`](https://docs.opencv.org/master/d4/d86/group__imgproc__filter.html#ga564869aa33e58769b4469101aac458f9) uses the median value of the neighbours in the kernel.

For this application, it makes the most sense to use the median blur, as that will allow the larger red part of the card to overtake the smaller white parts.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxCv.h"
#include "ofxGui.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  void mousePressed(int x, int y, int button);

  ofVideoGrabber grabber;
  ofImage processImg;

  ofxCv::ContourFinder contourFinder;

  ofParameter<ofColor> colorTarget;
  ofParameter<int> colorOffset;
  
  ofColor colorUnderMouse;

  ofParameter<float> minArea;
  ofParameter<float> maxArea;

  ofParameter<int> blurAmount;

  ofParameter<bool> debugProcess;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  // Setup the grabber.
  grabber.setup(640, 480);

  // Setup the contour finder and parameters.
  contourFinder.setUseTargetColor(true);

  colorTarget.set("Color Target", ofColor(255, 0, 0));
  colorOffset.set("Color Offset", 10, 0, 255);
  minArea.set("Min Area", 0.01f, 0, 0.5f);
  maxArea.set("Max Area", 0.05f, 0, 0.5f);
  blurAmount.set("Blur Amount", 0, 0, 100);
  debugProcess.set("Debug Process", false);

  // Setup the gui.
  guiPanel.setup("Color Tracker", "settings.json");
  guiPanel.add(colorTarget);
  guiPanel.add(colorOffset);
  guiPanel.add(minArea);
  guiPanel.add(maxArea);
  guiPanel.add(blurAmount);
  guiPanel.add(debugProcess);
}

void ofApp::update()
{
  grabber.update();
  if (grabber.isFrameNew())
  {
    processImg.setFromPixels(grabber.getPixels());

    // Filter the image.
    if (blurAmount > 0)
    {
      //ofxCv::blur(processImg, blurAmount);
      //ofxCv::GaussianBlur(processImg, blurAmount);
      ofxCv::medianBlur(processImg, blurAmount);
      processImg.update();
    }

    // Save the color of the pixel under the mouse.
    colorUnderMouse = processImg.getColor(ofGetMouseX(), ofGetMouseY());

    // Update parameters.
    contourFinder.setTargetColor(colorTarget, ofxCv::TRACK_COLOR_HSV);
    contourFinder.setThreshold(colorOffset);
    contourFinder.setMinAreaNorm(minArea);
    contourFinder.setMaxAreaNorm(maxArea);

    // Find contours.
    contourFinder.findContours(processImg);
  }
}

void ofApp::draw()
{
  ofSetColor(255);

  if (debugProcess)
  {
    // Draw the process image.
    processImg.draw(0, 0, ofGetWidth(), ofGetHeight());
  }
  else
  {
    // Draw the grabber image.
    grabber.draw(0, 0, ofGetWidth(), ofGetHeight());
  }

  // Draw the found contours.
  contourFinder.draw();

  // Draw the color under the mouse.
  ofPushStyle();
  ofSetColor(colorUnderMouse);
  ofDrawRectangle(ofGetMouseX() - 25, ofGetMouseY() - 25, 50, 50);
  ofNoFill();
  ofSetColor(colorUnderMouse.getInverted());
  ofDrawRectangle(ofGetMouseX() - 25, ofGetMouseY() - 25, 50, 50);
  ofPopStyle();

  // Draw the gui.
  guiPanel.draw();
}

void ofApp::mousePressed(int x, int y, int button)
{
  if (!guiPanel.getShape().inside(x, y))
  {
    // Track the color under the mouse.
    colorTarget = colorUnderMouse;
  }
}
```

### Dilate / Erode

[Dilation and erosion](https://docs.opencv.org/2.4/doc/tutorials/imgproc/erosion_dilatation/erosion_dilatation.html) are *morphology* based operations, meaning that they are based on shapes in the image. These tend to be used to remove noise, or to find features in an image like holes.

Dilation and erosion are counterparts of each other.

* [`dilate()`](https://docs.opencv.org/master/d4/d86/group__imgproc__filter.html#ga4ff0f3318642c4f469d0e11f242f3b6c) grows the bright regions of the image to take up more pixels.

{{< image src="https://docs.opencv.org/2.4/_images/Morphology_1_Tutorial_Theory_Dilatation_2.png" alt="Dilation" caption="[*Eroding and Dilating*](https://docs.opencv.org/2.4/doc/tutorials/imgproc/erosion_dilatation/erosion_dilatation.html)" width="600px" >}}

* [`erode()`](https://docs.opencv.org/master/d4/d86/group__imgproc__filter.html#gaeb1e0c1033e3f6b891a25d0511362aeb) grows the dark regions of the image to take up more pixels.

{{< image src="https://docs.opencv.org/2.4/_images/Morphology_1_Tutorial_Theory_Erosion_2.png" alt="Dilation" caption="[*Eroding and Dilating*](https://docs.opencv.org/2.4/doc/tutorials/imgproc/erosion_dilatation/erosion_dilatation.html)" width="600px" >}}

For this application, we should use erosion as we want the darker red parts of the card to overtake the brighter white parts.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxCv.h"
#include "ofxGui.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  void mousePressed(int x, int y, int button);

  ofVideoGrabber grabber;
  ofImage processImg;

  ofxCv::ContourFinder contourFinder;

  ofParameter<ofColor> colorTarget;
  ofParameter<int> colorOffset;
  
  ofColor colorUnderMouse;

  ofParameter<float> minArea;
  ofParameter<float> maxArea;

  ofParameter<int> blurAmount;
  ofParameter<int> erodeIterations;

  ofParameter<bool> debugProcess;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  // Setup the grabber.
  grabber.setup(640, 480);

  // Setup the contour finder and parameters.
  contourFinder.setUseTargetColor(true);

  colorTarget.set("Color Target", ofColor(255, 0, 0));
  colorOffset.set("Color Offset", 10, 0, 255);
  minArea.set("Min Area", 0.01f, 0, 0.5f);
  maxArea.set("Max Area", 0.05f, 0, 0.5f);
  blurAmount.set("Blur Amount", 0, 0, 100);
  erodeIterations.set("Erode Iterations", 0, 0, 10);
  debugProcess.set("Debug Process", false);

  // Setup the gui.
  guiPanel.setup("Color Tracker", "settings.json");
  guiPanel.add(colorTarget);
  guiPanel.add(colorOffset);
  guiPanel.add(minArea);
  guiPanel.add(maxArea);
  guiPanel.add(blurAmount);
  guiPanel.add(erodeIterations);
  guiPanel.add(debugProcess);
}

void ofApp::update()
{
  grabber.update();
  if (grabber.isFrameNew())
  {
    processImg.setFromPixels(grabber.getPixels());

    // Filter the image.
    if (blurAmount > 0)
    {
      //ofxCv::blur(processImg, blurAmount);
      //ofxCv::GaussianBlur(processImg, blurAmount);
      ofxCv::medianBlur(processImg, blurAmount);
    }
    if (erodeIterations > 0)
    {
      ofxCv::erode(processImg, erodeIterations.get());
    }
    processImg.update();

    // Save the color of the pixel under the mouse.
    colorUnderMouse = processImg.getColor(ofGetMouseX(), ofGetMouseY());

    // Update parameters.
    contourFinder.setTargetColor(colorTarget, ofxCv::TRACK_COLOR_HSV);
    contourFinder.setThreshold(colorOffset);
    contourFinder.setMinAreaNorm(minArea);
    contourFinder.setMaxAreaNorm(maxArea);

    // Find contours.
    contourFinder.findContours(processImg);
  }
}

void ofApp::draw()
{
  ofSetColor(255);

  if (debugProcess)
  {
    // Draw the process image.
    processImg.draw(0, 0, ofGetWidth(), ofGetHeight());
  }
  else
  {
    // Draw the grabber image.
    grabber.draw(0, 0, ofGetWidth(), ofGetHeight());
  }

  // Draw the found contours.
  contourFinder.draw();

  // Draw the color under the mouse.
  ofPushStyle();
  ofSetColor(colorUnderMouse);
  ofDrawRectangle(ofGetMouseX() - 25, ofGetMouseY() - 25, 50, 50);
  ofNoFill();
  ofSetColor(colorUnderMouse.getInverted());
  ofDrawRectangle(ofGetMouseX() - 25, ofGetMouseY() - 25, 50, 50);
  ofPopStyle();

  // Draw the gui.
  guiPanel.draw();
}

void ofApp::mousePressed(int x, int y, int button)
{
  if (!guiPanel.getShape().inside(x, y))
  {
    // Track the color under the mouse.
    colorTarget = colorUnderMouse;
  }
}
```

## Tracking

Blob tracking means to follow and track objects over time. Instead of interpreting each frame as a unique "event", we can compare the current blobs detected to previous ones and see if there are correspondences. By tracking blobs over time, we can assign them parameters like an age, a direction of movement, a velocity, and use those parameters to build rich interactive applications.

<figure style="width:600px;height:400px;display:block;margin:0 auto;" markdown="1">
<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/137879996" style="position:absolute;top:0;left:0;width:100%;height:100%;" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>
<figcaption><i><a href="https://vimeo.com/137879996">collimation</a> from <a href="https://vimeo.com/user22070294">Brad Todd</a> on <a href="https://vimeo.com">Vimeo</a>.</i></figcaption>
</figure>

Tracking is not part of the OpenCV library but it is such a common operation to perform post contour finding that a set of tracking functions are included with `ofxCv`.

`ofxCv::Tracker` is a powerful blob tracker that can be used on its own or with `ofxCv::ContourFinder`. In fact, `ofxCv::ContourFinder` has a tracker embedded in it which can be accessed using `ofxCv::ContourFinder.getTracker()`.

Tracked blobs are identified using a label. If two blobs from different frames have the same label, then they are considered the same.

The tracker takes in parameters to set how it operates.

* *Persistence* is the amount of time a blob can disappear before it is actually considered dead.
  * This is set in frames.
  * If it is set to `0` then a blob will stop being tracked as soon as it disappears.
  * If it is set higher, for example to `15`, then a blob can disappear for up to 15 frames and then reappear and keep its previous label.
  * This can be helpful if the video feed is not stable and blobs seem to flicker on and off.
* *Max distance* is the maximum distance a blob can travel between two frames.
  * This is set in pixels.
  * If a blob is less than the max distance away from a previous blob, it is considered one and the same and keeps its label.
  * If a blob is further away than max distance from any previous blobs, it is considered new and gets a new label.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxCv.h"
#include "ofxGui.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  void mousePressed(int x, int y, int button);

  ofVideoGrabber grabber;
  ofImage processImg;

  ofxCv::ContourFinder contourFinder;

  ofParameter<ofColor> colorTarget;
  ofParameter<int> colorOffset;
  
  ofColor colorUnderMouse;

  ofParameter<float> minArea;
  ofParameter<float> maxArea;

  ofParameter<int> blurAmount;
  ofParameter<int> erodeIterations;

  ofParameter<int> persistence;
  ofParameter<float> maxDistance;

  ofParameter<bool> showLabels;
  ofParameter<bool> debugProcess;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  // Setup the grabber.
  grabber.setup(640, 480);

  // Setup the contour finder and parameters.
  contourFinder.setUseTargetColor(true);

  colorTarget.set("Color Target", ofColor(255, 0, 0));
  colorOffset.set("Color Offset", 10, 0, 255);
  minArea.set("Min Area", 0.01f, 0, 0.5f);
  maxArea.set("Max Area", 0.05f, 0, 0.5f);
  blurAmount.set("Blur Amount", 0, 0, 100);
  erodeIterations.set("Erode Iterations", 0, 0, 10);
  persistence.set("Persistence", 15, 0, 60);
  maxDistance.set("Max Distance", 64, 0, 640);
  showLabels.set("Show Labels", false);
  debugProcess.set("Debug Process", false);

  // Setup the gui.
  guiPanel.setup("Color Tracker", "settings.json");
  guiPanel.add(colorTarget);
  guiPanel.add(colorOffset);
  guiPanel.add(minArea);
  guiPanel.add(maxArea);
  guiPanel.add(blurAmount);
  guiPanel.add(erodeIterations);
  guiPanel.add(persistence);
  guiPanel.add(maxDistance);
  guiPanel.add(showLabels);
  guiPanel.add(debugProcess);
}

void ofApp::update()
{
  grabber.update();
  if (grabber.isFrameNew())
  {
    processImg.setFromPixels(grabber.getPixels());

    // Filter the image.
    if (blurAmount > 0)
    {
      //ofxCv::blur(processImg, blurAmount);
      //ofxCv::GaussianBlur(processImg, blurAmount);
      ofxCv::medianBlur(processImg, blurAmount);
    }
    if (erodeIterations > 0)
    {
      ofxCv::erode(processImg, erodeIterations.get());
    }
    processImg.update();

    // Save the color of the pixel under the mouse.
    colorUnderMouse = processImg.getColor(ofGetMouseX(), ofGetMouseY());

    // Update parameters.
    contourFinder.setTargetColor(colorTarget, ofxCv::TRACK_COLOR_HSV);
    contourFinder.setThreshold(colorOffset);
    contourFinder.setMinAreaNorm(minArea);
    contourFinder.setMaxAreaNorm(maxArea);
    contourFinder.getTracker().setPersistence(persistence);
    contourFinder.getTracker().setMaximumDistance(maxDistance);

    // Find contours.
    contourFinder.findContours(processImg);
  }
}

void ofApp::draw()
{
  ofSetColor(255);

  if (debugProcess)
  {
    // Draw the process image.
    processImg.draw(0, 0, ofGetWidth(), ofGetHeight());
  }
  else
  {
    // Draw the grabber image.
    grabber.draw(0, 0, ofGetWidth(), ofGetHeight());
  }

  // Draw the found contours.
  contourFinder.draw();

  if (showLabels)
  {
    ofxCv::RectTracker& tracker = contourFinder.getTracker();

    ofSetColor(255);
    for (int i = 0; i < contourFinder.size(); i++) 
    {
      ofPoint center = ofxCv::toOf(contourFinder.getCenter(i));
      int label = contourFinder.getLabel(i);
      string msg = ofToString(label) + ":" + ofToString(tracker.getAge(label));
      ofDrawBitmapString(msg, center.x, center.y);
      ofVec2f velocity = ofxCv::toOf(contourFinder.getVelocity(i));
      ofDrawLine(center.x, center.y, center.x + velocity.x, center.y + velocity.y);
    }
  }

  // Draw the color under the mouse.
  ofPushStyle();
  ofSetColor(colorUnderMouse);
  ofDrawRectangle(ofGetMouseX() - 25, ofGetMouseY() - 25, 50, 50);
  ofNoFill();
  ofSetColor(colorUnderMouse.getInverted());
  ofDrawRectangle(ofGetMouseX() - 25, ofGetMouseY() - 25, 50, 50);
  ofPopStyle();

  // Draw the gui.
  guiPanel.draw();
}

void ofApp::mousePressed(int x, int y, int button)
{
  if (!guiPanel.getShape().inside(x, y))
  {
    // Track the color under the mouse.
    colorTarget = colorUnderMouse;
  }
}
```
