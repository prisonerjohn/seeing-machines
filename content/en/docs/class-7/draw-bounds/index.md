---
title: "Draw Bounds"
description: ""
lead: ""
date: 2022-11-04T15:46:22-04:00
lastmod: 2022-11-04T15:46:22-04:00
draft: false
images: []
menu:
  docs:
    parent: "class-7"
    identifier: "draw-bounds"
weight: 830
toc: true
---

Most of the example apps we have built so far have had the same window size as the video grabber or depth camera size.

```cpp
ofSetWindowShape(640, 480);
grabber.setup(640, 480);
```

This is convenient because window coordinates match pixel coordinates, and we can easily do things like sampling using the mouse position without too much trouble.

However, it is also limiting as we can't define the window dimensions we want. We will usually want the video to take up the entire screen, especially for presentation purposes, so we need to get comfortable with image scaling.

## Scale and Aspect Ratio

Most OF classes with a draw method that takes an `x` and `y` usually can also accept a `width` and `height`. We could just set this to the window's width and height to have the image take up the entire available space.

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
  void keyPressed(int key);

  ofVideoGrabber grabber;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  grabber.setup(640, 480);
}

void ofApp::update()
{
  grabber.update();
}

void ofApp::draw()
{
  // Scale and stretch.
  grabber.draw(0, 0, ofGetWidth(), ofGetHeight());
}

void ofApp::keyPressed(int key)
{
  if (key == 'f')
  {
    ofToggleFullscreen();
  }
  else if (key == 'w')
  {
    ofSetFullscreen(false);
    ofSetWindowShape(grabber.getWidth(), grabber.getHeight());
  }
}
```

This is quick to implement but will stretch the image if its aspect ratio is different from the window's aspect ratio. If we want to keep the aspect ratio the same, we need to do a bit of math.

We can choose to match either the width or height of the image with the width or height of the window. We then compute the other dimension using the image's aspect ratio to avoid any stretching.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  grabber.setup(640, 480);
}

void ofApp::update()
{
  grabber.update();
}

void ofApp::draw()
{
  // Scale and keep aspect ratio.
  float grabberRatio = grabber.getWidth() / grabber.getHeight();
  float windowRatio = ofGetWidth() / (float)ofGetHeight();

  // Fill width.
  float drawWidth = ofGetWidth();
  float drawHeight = drawWidth / grabberRatio;
  // OR
  // Fill height.
  //float drawHeight = ofGetHeight();
  //float drawWidth = drawHeight * grabberRatio;

  // Center in window.
  float drawX = (ofGetWidth() - drawWidth) / 2.0f;
  float drawY = (ofGetHeight() - drawHeight) / 2.0f;

  grabber.draw(drawX, drawY, drawWidth, drawHeight);
}

void ofApp::keyPressed(int key)
{
  if (key == 'f')
  {
    ofToggleFullscreen();
  }
  else if (key == 'w')
  {
    ofSetFullscreen(false);
    ofSetWindowShape(grabber.getWidth(), grabber.getHeight());
  }
}
```

## ofRectangle

[`ofRectangle`](https://openframeworks.cc/documentation/types/ofRectangle/) is a very versatile openFrameworks type. It has a position and dimensions (`x`, `y`, `width`, `height`) and many useful functions for scaling, moving, drawing, checking inclusion, etc.

We can use the [`ofRectangle.scaleTo()`](https://openframeworks.cc/documentation/types/ofRectangle/#show_scaleTo) to calculate image draw bounds while keeping the original aspect ratio. We do this by creating a new `ofRectangle` using the source image, then scaling it to a second `ofRectangle` representing the window dimensions. This can be done manually with

```cpp
ofRectangle windowBounds = ofRectangle(0, 0, ofGetWidth(), ofGetHeight());
```

or automatically by calling [`ofGetCurrentViewport()`](https://openframeworks.cc/documentation/graphics/ofGraphics/#!show_ofGetCurrentViewport).

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  grabber.setup(640, 480);
}

void ofApp::update()
{
  grabber.update();
}

void ofApp::draw()
{
  // Scale using ofRectangle.
  ofRectangle drawBounds;
  drawBounds.set(0, 0, grabber.getWidth(), grabber.getHeight());
  // Scale to fit, keeping the entire image visible.
  drawBounds.scaleTo(ofGetCurrentViewport(), OF_SCALEMODE_FIT);
  // OR
  // Scale to fill, covering the entire window.
  //drawBounds.scaleTo(ofGetCurrentViewport(), OF_SCALEMODE_FILL);

  grabber.draw(drawBounds);
}

void ofApp::keyPressed(int key)
{
  if (key == 'f')
  {
    ofToggleFullscreen();
  }
  else if (key == 'w')
  {
    ofSetFullscreen(false);
    ofSetWindowShape(grabber.getWidth(), grabber.getHeight());
  }
}
```

## Transformation Matrix

Some classes do not allow parameters in their `draw()` function, and will always draw their content at `(0, 0)` and native resolution.

In order to change these properties, we need to modify the surface we are drawing on, and not the object itself. We can think of this as keeping a pencil in place and moving the paper under it to draw (instead of keeping the paper still and moving the pencil over it).

These types of operations are called matrix transformations. We've already encountered them before when using [`ofScale()`](https://openframeworks.cc/documentation/graphics/ofGraphics/#!show_ofScale) and [`ofTranslate()`](https://openframeworks.cc/documentation/graphics/ofGraphics/#!show_ofTranslate) so this should feel familiar.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  grabber.setup(640, 480);
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
  // OR
  // Fill height.
  //float scaleRatio = ofGetHeight() / grabber.getHeight();
  //float drawY = 0;
  //float drawWidth = grabber.getWidth() * scaleRatio;
  //float drawX = (ofGetWidth() - drawWidth) / 2.0f;

  ofTranslate(drawX, drawY);
  ofScale(scaleRatio);

  grabber.draw(0, 0);
}

void ofApp::keyPressed(int key)
{
  if (key == 'f')
  {
    ofToggleFullscreen();
  }
  else if (key == 'w')
  {
    ofSetFullscreen(false);
    ofSetWindowShape(grabber.getWidth(), grabber.getHeight());
  }
}
```

## Mapping

Even though we are changing the resolution of the drawn image, we are not changing the actual dimensions of the pixel data. A `640x480 px` frame still has `640x480 px` of data even when it is drawn at `1280x960 px`.

We need to stay mindful of this when doing color look-ups, and make sure that we always remain in the image space when doing so.

openFrameworks includes a useful [`ofMap()`](https://openframeworks.cc/documentation/math/ofMath/#!show_ofMap) function that easily maps values from one range to another. We can use this in the following example, where we use the mouse to pick the pixel color under the cursor of a scaled image.

Note that we set the optional "clamp" parameter of `ofMap` to true to ensure that our pixel coordinate always stays within bounds, even when the mouse exits the window.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  grabber.setup(640, 480);
}

void ofApp::update()
{
  grabber.update();
}

void ofApp::draw()
{
  // Scale using ofRectangle
  ofRectangle drawBounds;
  drawBounds.set(0, 0, grabber.getWidth(), grabber.getHeight());
  // Scale to fill, covering the entire window.
  drawBounds.scaleTo(ofGetCurrentViewport(), OF_SCALEMODE_FILL);
  // Draw the grabber image.
  grabber.draw(drawBounds);

  // Get a reference to the image pixels.
  ofPixels& grabberPix = grabber.getPixels();
  // Get the color value under the mouse.
  // Remap from draw space to pixel space.
  int sampleX = ofMap(ofGetMouseX(), drawBounds.x, drawBounds.width, 0, grabber.getWidth(), true);
  int sampleY = ofMap(ofGetMouseY(), drawBounds.y, drawBounds.height, 0, grabber.getHeight(), true);
  ofColor color = grabberPix.getColor(sampleX, sampleY);

  // Draw a rectangle under the mouse using the pixel color.
  ofFill();
  ofSetColor(color);
  ofDrawRectangle(ofGetMouseX() - 25, ofGetMouseY() - 25, 50, 50);
  // Add an outline so we can see the rectangle better.
  ofNoFill();
  ofSetColor(0);
  ofDrawRectangle(ofGetMouseX() - 25, ofGetMouseY() - 25, 50, 50);
  ofSetColor(255);
}

void ofApp::keyPressed(int key)
{
  if (key == 'f')
  {
    ofToggleFullscreen();
  }
  else if (key == 'w')
  {
    ofSetFullscreen(false);
    ofSetWindowShape(grabber.getWidth(), grabber.getHeight());
  }
}
```
