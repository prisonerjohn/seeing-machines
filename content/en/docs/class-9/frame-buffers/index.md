---
title: "Frame Buffers"
description: ""
lead: ""
date: 2022-11-13T09:51:58-05:00
lastmod: 2022-11-13T09:51:58-05:00
draft: true
images: []
menu:
  docs:
    parent: "class-8"
    identifier: "frame-buffers"
weight: 920
toc: true
---

When we draw something in OF, whether it's a circle or an image or a 3D shape, it gets drawn to the screen by default. But we don't need to necessarily draw to the screen; we can also draw to an offscreen location called a *framebuffer*.

A framebuffer object (or FBO) is a simple OpenGL object that lives in graphics memory that we can draw into. We can think of it as a "virtual window"; anything we can draw on screen we can draw inside this window.

## ofFbo

In openFrameworks, the [`ofFbo`](https://openframeworks.cc/documentation/gl/ofFbo/) object is a wrapper around a framebuffer.

* We first need to create the `ofFbo` using [`allocate()`](https://openframeworks.cc/documentation/gl/ofFbo/#show_allocate).
* We can turn it on or off using the [`begin()`](https://openframeworks.cc/documentation/gl/ofFbo/#show_begin) and [`end()`](https://openframeworks.cc/documentation/gl/ofFbo/#show_end) functions.
  * Anything we draw between the calls to `begin()` and `end()` will be drawn inside that `ofFbo`.
* An `ofFbo` actually renders to an `ofTexture`, and we can generally use an `ofFbo` anywhere we would use a texture.
  * We can draw the FBO directly to the screen using [`draw()`](https://openframeworks.cc/documentation/gl/ofFbo/#show_draw).
  * We can retrieve the texture using [`getTexture()`](https://openframeworks.cc/documentation/gl/ofFbo/#show_getTexture).

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  void keyPressed(int key);

  ofFbo canvasFbo;

  ofParameter<ofColor> tintColor;
  ofParameter<bool> clearFbo;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 720);

  // Allocate the frame buffer.
  ofFboSettings settings;
  settings.width = 640;
  settings.height = 360;
  canvasFbo.allocate(settings);

  // Setup the parameters.
  tintColor.set("Tint Color", ofColor(0, 255, 0));
  clearFbo.set("Clear Background", false);

  // Setup the gui.
  guiPanel.setup("Fbo Draw", "settings.json");
  guiPanel.add(tintColor);
  guiPanel.add(clearFbo);
}

void ofApp::update()
{
  canvasFbo.begin();
  {
    if (clearFbo)
    {
      // Clear the background.
      ofBackground(0);
      clearFbo = false;
    }

    if (ofGetMousePressed() && !guiPanel.getShape().inside(ofGetMouseX(), ofGetMouseY()))
    {
      // Draw a circle if the mouse is pressed and not over the GUI.
      ofSetColor(255);
      ofDrawCircle(ofGetMouseX(), ofGetMouseY(), 20);
    }
  }
  canvasFbo.end();
}

void ofApp::draw()
{
  // Draw the canvas above with no tint.
  ofSetColor(255);
  canvasFbo.draw(0, 0);

  // Draw the canvas below with a tint color.
  ofSetColor(tintColor);
  canvasFbo.draw(0, 360);

  guiPanel.draw();
}

void ofApp::keyPressed(int key)
{
  if (key == ' ')
  {
    clearFbo = true;
  }
}
```

The `ofFbo` object lives on the GPU. The data is available as an `ofTexture` only. `ofTexture` data can be read back to the CPU using the [`readToPixels()`](https://openframeworks.cc/documentation/gl/ofFbo/#show_readToPixels) function. This will store the data in the passed `ofPixels` object, which can then be used anywhere you would use any other pixel data, like with OpenCV. Note however that most systems are set up and optimized for CPU to GPU data transfer, not the other way around. While GPU to CPU does work, it should be used sparingly as it could slow down your application.

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

  void keyPressed(int key);

  ofxCv::ContourFinder contourFinder;
  
  ofFbo canvasFbo;
  ofFbo visionFbo;

  ofPixels canvasPixels;

  ofParameter<ofColor> tintColor;
  ofParameter<bool> clearFbo;

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
    ofSetWindowShape(640, 720);

    // Allocate the frame buffers.
    ofFboSettings settings;
    settings.width = 640;
    settings.height = 360;
    canvasFbo.allocate(settings);
    visionFbo.allocate(settings);

    // Setup the parameters.
    tintColor.set("Tint Color", ofColor(0, 255, 0));
    clearFbo.set("Clear Background", false);

    minArea.set("Min Area", 0.01f, 0, 0.5f);
    maxArea.set("Max Area", 0.05f, 0, 0.5f);

    // Setup the gui.
    guiPanel.setup("Fbo Draw", "settings.json");
    guiPanel.add(tintColor);
    guiPanel.add(clearFbo);
    guiPanel.add(minArea);
    guiPanel.add(maxArea);
}

void ofApp::update()
{
  canvasFbo.begin();
  {
    if (clearFbo)
    {
      // Clear the background.
      ofBackground(0);
      clearFbo = false;
    }

    if (ofGetMousePressed() && !guiPanel.getShape().inside(ofGetMouseX(), ofGetMouseY()))
    {
      // Draw a circle if the mouse is pressed and not over the GUI.
      ofSetColor(255);
      ofDrawCircle(ofGetMouseX(), ofGetMouseY(), 20);
    }
  }
  canvasFbo.end();

  // Download the FBO data as pixels.
  canvasFbo.readToPixels(canvasPixels);

  // Find contours.
  contourFinder.setMinAreaNorm(minArea);
  contourFinder.setMaxAreaNorm(maxArea);
  contourFinder.findContours(canvasPixels);

  // Draw the result offscreen.
  visionFbo.begin();
  {
    ofBackground(127);

    contourFinder.draw();
  }
  visionFbo.end();
}

void ofApp::draw()
{
  // Draw the canvas above with no tint.
  ofSetColor(255);
  canvasFbo.draw(0, 0);

  // Draw the contours below with a tint color.
  ofSetColor(tintColor);
  visionFbo.draw(0, 360);

  guiPanel.draw();
}

void ofApp::keyPressed(int key)
{
  if (key == ' ')
  {
    clearFbo = true;
  }
}
```

An `ofFbo` is a good way to position and scale an `ofTexture` before drawing it. Everything can be drawn inside the FBO texture at native resolution, then transformed as a whole. This is particularly useful for drawing objects that don't have any scaling built-in, like `ofxCv::ContourFinder` for example.

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

  void keyPressed(int key);

  void toggleFullscreen(bool& fullscreen);

  ofxCv::ContourFinder contourFinder;
  
  ofFbo canvasFbo;
  ofFbo visionFbo;

  ofPixels canvasPixels;

  ofParameter<bool> fullscreen;

  ofParameter<ofColor> tintColor;
  ofParameter<bool> clearFbo;

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
  ofSetWindowShape(640, 720);

  // Allocate the frame buffers.
  ofFboSettings settings;
  settings.width = 640;
  settings.height = 360;
  canvasFbo.allocate(settings);
  visionFbo.allocate(settings);

  // Setup the parameters.
  fullscreen.set("Fullscreen", false);
  fullscreen.addListener(this, &ofApp::toggleFullscreen);

  tintColor.set("Tint Color", ofColor(0, 255, 0));
  clearFbo.set("Clear Background", false);

  minArea.set("Min Area", 0.01f, 0, 0.5f);
  maxArea.set("Max Area", 0.05f, 0, 0.5f);

  // Setup the gui.
  guiPanel.setup("Fbo Draw", "settings.json");
  guiPanel.add(fullscreen);
  guiPanel.add(tintColor);
  guiPanel.add(clearFbo);
  guiPanel.add(minArea);
  guiPanel.add(maxArea);
}

void ofApp::update()
{
  canvasFbo.begin();
  {
    if (clearFbo)
    {
      // Clear the background.
      ofBackground(0);
      clearFbo = false;
    }

    if (ofGetMousePressed() && !guiPanel.getShape().inside(ofGetMouseX(), ofGetMouseY()))
    {
      // Draw a circle if the mouse is pressed and not over the GUI.
      ofSetColor(255);
      ofDrawCircle(ofGetMouseX(), ofGetMouseY(), 20);
    }
  }
  canvasFbo.end();

  // Download the FBO data as pixels.
  canvasFbo.readToPixels(canvasPixels);

  // Find contours.
  contourFinder.setMinAreaNorm(minArea);
  contourFinder.setMaxAreaNorm(maxArea);
  contourFinder.findContours(canvasPixels);

  // Draw the result offscreen.
  visionFbo.begin();
  {
    ofBackground(127);

    canvasFbo.draw(0, 0);

    contourFinder.draw();
  }
  visionFbo.end();
}

void ofApp::draw()
{
  if (fullscreen)
  {
    // Draw the contours fullscreen with no tint.
    ofRectangle drawRect = ofRectangle(0, 0, visionFbo.getWidth(), visionFbo.getHeight());
    drawRect.scaleTo(ofGetCurrentViewport());
    ofSetColor(255);
    visionFbo.draw(drawRect);
  }
  else
  {
    // Draw the canvas above with no tint.
    ofSetColor(255);
    canvasFbo.draw(0, 0);

    // Draw the contours below with a tint color.
    ofSetColor(tintColor);
    visionFbo.draw(0, 360);
  }

  guiPanel.draw();
}

void ofApp::keyPressed(int key)
{
  if (key == ' ')
  {
    clearFbo = true;
  }
  else if (key == OF_KEY_TAB)
  {
    fullscreen = !fullscreen;
    ofSetFullscreen(fullscreen);
  }
}

void ofApp::toggleFullscreen(bool& fullscreen)
{
  ofSetFullscreen(fullscreen);
}
```

## 3D

FBOs are also quite useful when working in 3D space, as they can provide a "window" into the 3D world. This window can then be manipulated just like any other `ofTexture`.

The following example uses a RealSense to generate a point cloud, and an `ofEasyCam` to display it in 3D.

```cpp
camera.begin();
ofEnableDepthTest();
ofPushMatrix();

// Adjust points to match OF 3D space.
ofScale(100);
ofRotateXDeg(180);
rsDevice->getColorTex().bind();
rsDevice->getPointsMesh().draw();
rsDevice->getColorTex().unbind();

ofPopMatrix();
ofDisableDepthTest();
camera.end();
```

By default, the camera will take up the entire window resolution to draw the world in. This is less than ideal if we want to draw other elements in the window.

{{< image src="fbo-overlap.png" alt="FBO Overlap" caption="*FBO Overlap*" width="600px" >}}

We can pass an `ofRectangle` to [`ofCamera.begin()`](https://openframeworks.cc/documentation/3d/ofCamera/#show_begin) to tell it how to delimit the space it renders in.

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

  void keyPressed(int key);

  void deviceAdded(string& serialNumber);

  ofxRealSense2::Context rsContext;

  ofImage thresholdImg;

  ofxCv::ContourFinder contourFinder;
  
  ofEasyCam camera;

  ofFbo visionFbo;

  ofParameter<float> nearThreshold;
  ofParameter<float> farThreshold;

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
  // Texture coordinates from RealSense are normalized (between 0-1).
  // This call normalizes all OF texture coordinates so that they match.
  ofDisableArbTex();

  ofSetWindowShape(1280, 720);

  // Start the RealSense context.
  // Devices are added in the deviceAdded() callback function.
  ofAddListener(rsContext.deviceAddedEvent, this, &ofApp::deviceAdded);
  rsContext.setup(false);

  // Allocate the frame buffers.
  ofFboSettings settings;
  settings.width = 640;
  settings.height = 360;
  visionFbo.allocate(settings);

  // Setup the parameters.
  nearThreshold.set("Near Threshold", 0.01f, 0.0f, 0.1f);
  farThreshold.set("Far Threshold", 0.02f, 0.0f, 0.1f);
  minArea.set("Min Area", 0.01f, 0, 0.5f);
  maxArea.set("Max Area", 0.05f, 0, 0.5f);

  // Setup the gui.
  guiPanel.setup("Depth Threshold", "settings.json");
  guiPanel.add(nearThreshold);
  guiPanel.add(farThreshold);
  guiPanel.add(minArea);
  guiPanel.add(maxArea);
}

void ofApp::deviceAdded(string& serialNumber)
{
  ofLogNotice(__FUNCTION__) << "Starting device " << serialNumber;
  auto device = rsContext.getDevice(serialNumber);
  device->enableDepth();
  device->enableColor();
  device->enablePoints();
  device->startPipeline();

  // Uncomment this to add the device specific settings to the GUI.
  //guiPanel.add(device->params);
}

void ofApp::update()
{
  rsContext.update();

  shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice && rsDevice->isFrameNew())
  {
    // Threshold the depth.
    ofFloatPixels rawDepthPix = rsDevice->getRawDepthPix();
    ofFloatPixels thresholdNear, thresholdFar, thresholdResult;
    ofxCv::threshold(rawDepthPix, thresholdNear, nearThreshold);
    ofxCv::threshold(rawDepthPix, thresholdFar, farThreshold, true);
    ofxCv::bitwise_and(thresholdNear, thresholdFar, thresholdResult);
    thresholdImg.setFromPixels(thresholdResult);

    // Find contours.
    contourFinder.setMinAreaNorm(minArea);
    contourFinder.setMaxAreaNorm(maxArea);
    contourFinder.findContours(thresholdImg);

    // Draw CV operations.
    visionFbo.begin();
    {
      // Draw the threshold background.
      ofSetColor(255);
      thresholdImg.draw(0, 0);

      // Draw the contours over it.
      ofSetColor(0, 255, 0);
      contourFinder.draw();
    }
    visionFbo.end();
  }
}

void ofApp::draw()
{
  shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    // 2x2 window.
    int frameWidth = ofGetWidth() / 2;
    int frameHeight = ofGetHeight() / 2;

    rsDevice->getDepthTex().draw(0, 0, frameWidth, frameHeight);
    rsDevice->getColorTex().draw(0, frameHeight, frameWidth, frameHeight);
    visionFbo.draw(frameWidth, 0, frameWidth, frameHeight);

    camera.begin(ofRectangle(frameWidth, frameHeight, frameWidth, frameHeight));
    ofEnableDepthTest();
    ofPushMatrix();

    // Adjust points to match OF 3D space.
    ofScale(100);
    ofRotateXDeg(180);

    rsDevice->getColorTex().bind();
    rsDevice->getPointsMesh().draw();
    rsDevice->getColorTex().unbind();

    ofPopMatrix();
    ofDisableDepthTest();
    camera.end();
  }

  // Draw the gui.
  guiPanel.draw();
}

void ofApp::keyPressed(int key)
{
  if (key == OF_KEY_TAB)
  {
    ofToggleFullscreen();
  }
}
```

While this works, we are still drawing directly in the app window, and would be unable to manipulate the camera render as a texture. A better approach is to draw the camera output directly in an `ofFbo`. Because we're using the entire frame buffer resolution for our drawing, we can revert to `ofCamera.begin()` without passing it a viewport.

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

  void keyPressed(int key);

  void deviceAdded(string& serialNumber);

  ofxRealSense2::Context rsContext;

  ofImage thresholdImg;

  ofxCv::ContourFinder contourFinder;
  
  ofEasyCam camera;

  ofFbo visionFbo;
  ofFbo pointsFbo;

  ofParameter<float> nearThreshold;
  ofParameter<float> farThreshold;

  ofParameter<float> minArea;
  ofParameter<float> maxArea;

  ofParameter<int> drawMode;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  // Texture coordinates from RealSense are normalized (between 0-1).
  // This call normalizes all OF texture coordinates so that they match.
  ofDisableArbTex();

  ofSetWindowShape(1280, 720);

  // Start the RealSense context.
  // Devices are added in the deviceAdded() callback function.
  ofAddListener(rsContext.deviceAddedEvent, this, &ofApp::deviceAdded);
  rsContext.setup(false);

  // Allocate the frame buffers.
  ofFboSettings settings;
  settings.width = 640;
  settings.height = 360;
  visionFbo.allocate(settings);
  pointsFbo.allocate(settings);

  // Setup the parameters.
  nearThreshold.set("Near Threshold", 0.01f, 0.0f, 0.1f);
  farThreshold.set("Far Threshold", 0.02f, 0.0f, 0.1f);
  minArea.set("Min Area", 0.01f, 0, 0.5f);
  maxArea.set("Max Area", 0.05f, 0, 0.5f);
  drawMode.set("Draw Mode", 0, 0, 4);

  // Setup the gui.
  guiPanel.setup("Depth Threshold", "settings.json");
  guiPanel.add(nearThreshold);
  guiPanel.add(farThreshold);
  guiPanel.add(minArea);
  guiPanel.add(maxArea);
  guiPanel.add(drawMode);
}

void ofApp::deviceAdded(string& serialNumber)
{
  ofLogNotice(__FUNCTION__) << "Starting device " << serialNumber;
  auto device = rsContext.getDevice(serialNumber);
  device->enableDepth();
  device->enableColor();
  device->enablePoints();
  device->startPipeline();

  // Uncomment this to add the device specific settings to the GUI.
  //guiPanel.add(device->params);
}

void ofApp::update()
{
  rsContext.update();

  shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice && rsDevice->isFrameNew())
  {
    // Threshold the depth.
    ofFloatPixels rawDepthPix = rsDevice->getRawDepthPix();
    ofFloatPixels thresholdNear, thresholdFar, thresholdResult;
    ofxCv::threshold(rawDepthPix, thresholdNear, nearThreshold);
    ofxCv::threshold(rawDepthPix, thresholdFar, farThreshold, true);
    ofxCv::bitwise_and(thresholdNear, thresholdFar, thresholdResult);
    thresholdImg.setFromPixels(thresholdResult);

    // Find contours.
    contourFinder.setMinAreaNorm(minArea);
    contourFinder.setMaxAreaNorm(maxArea);
    contourFinder.findContours(thresholdImg);

    // Draw CV operations.
    visionFbo.begin();
    {
      // Draw the threshold background.
      ofSetColor(255);
      thresholdImg.draw(0, 0);

      // Draw the contours over it.
      ofSetColor(0, 255, 0);
      contourFinder.draw();
    }
    visionFbo.end();

    // Draw 3D world.
    pointsFbo.begin();
    {
      ofBackground(0);

      camera.begin();
      ofEnableDepthTest();
      ofPushMatrix();

      // Adjust points to match OF 3D space.
      ofScale(1000);
      ofRotateXDeg(180);

      rsDevice->getColorTex().bind();
      rsDevice->getPointsMesh().draw();
      rsDevice->getColorTex().unbind();
      
      ofPopMatrix();
      ofDisableDepthTest();
      camera.end();
    }
    pointsFbo.end();
  }
}

void ofApp::draw()
{
  shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    if (drawMode == 0)
    {
      // 2x2 window.
      int frameWidth = ofGetWidth() / 2;
      int frameHeight = ofGetHeight() / 2;

      rsDevice->getDepthTex().draw(0, 0, frameWidth, frameHeight);
      rsDevice->getColorTex().draw(0, frameHeight, frameWidth, frameHeight);
      visionFbo.draw(frameWidth, 0, frameWidth, frameHeight);
      pointsFbo.draw(frameWidth, frameHeight, frameWidth, frameHeight);
    }
    else if(drawMode == 1)
    {
      // Draw fullscreen depth.
      rsDevice->getDepthTex().draw(ofGetCurrentViewport());
    }
    else if (drawMode == 2)
    {
      // Draw fullscreen color.
      rsDevice->getColorTex().draw(ofGetCurrentViewport());
    }
    else if (drawMode == 3)
    {
      // Draw fullscreen vision.
      visionFbo.draw(ofGetCurrentViewport());
    }
    else if (drawMode == 4)
    {
      // Draw fullscreen points.
      pointsFbo.draw(ofGetCurrentViewport());
    }
  }

  // Draw the gui.
  guiPanel.draw();
}

void ofApp::keyPressed(int key)
{
  if (key == OF_KEY_TAB)
  {
    ofToggleFullscreen();
  }
  else if (key == ' ')
  {
    drawMode = (drawMode + 1) % 5;
  }
}
```

You might notice that the camera controls are acting a little weird. This is because the `ofEasyCam` thinks that it's drawing in the bounds `(0, 0) to (640, 360)` as that is the size of the `ofFbo`, and it therefore limits the mouse controls to that area of the window. This can be adjusted by giving `ofEasyCam` actual bounds rectangle that the world is being drawn in using [`ofEasyCam.setControlArea()`](https://openframeworks.cc/documentation/3d/ofEasyCam/#show_setControlArea). This should make mouse control feel more natural.

```cpp
// ofApp.cpp

// ...

void ofApp::draw()
{
  shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    ofRectangle camDrawRect;

    if (drawMode == 0)
    {
      // 2x2 window.
      int frameWidth = ofGetWidth() / 2;
      int frameHeight = ofGetHeight() / 2;

      rsDevice->getDepthTex().draw(0, 0, frameWidth, frameHeight);
      rsDevice->getColorTex().draw(0, frameHeight, frameWidth, frameHeight);
      visionFbo.draw(frameWidth, 0, frameWidth, frameHeight);

      camDrawRect = ofRectangle(frameWidth, frameHeight, frameWidth, frameHeight);
      pointsFbo.draw(camDrawRect);
    }
    else if(drawMode == 1)
    {
      // Draw fullscreen depth.
      rsDevice->getDepthTex().draw(ofGetCurrentViewport());
    }
    else if (drawMode == 2)
    {
      // Draw fullscreen color.
      rsDevice->getColorTex().draw(ofGetCurrentViewport());
    }
    else if (drawMode == 3)
    {
      // Draw fullscreen vision.
      visionFbo.draw(ofGetCurrentViewport());
    }
    else if (drawMode == 4)
    {
      // Draw fullscreen points.
      camDrawRect = ofGetCurrentViewport();
      pointsFbo.draw(camDrawRect);
    }

    camera.setControlArea(camDrawRect);
  }

  // Draw the gui.
  guiPanel.draw();
}

// ...
```

{{< image src="fbo-quads.png" alt="FBO Quads" caption="*FBO Quads*" width="600px" >}}

## Multi-window

When working in installation settings, we will usually have a multi-display setup. A common configuration is to have one monitor for controls and preview windows, and another display (either a large screen monitor or projector) for the main content. openFrameworks makes working with multiple windows easy, and includes some examples in `/path/to/OF/examples/windowing` demonstrating how this works.

The important thing to remember is that this needs to be configured before the `ofApp` starts running, therefore in the `main()` function found in `main.cpp`.

```cpp
// main.cpp
#include "ofApp.h"

int main()
{
  ofGLFWWindowSettings settings;

  settings.setSize(1280, 720);
  settings.setPosition(glm::vec2(0, 0));
  settings.resizable = true;
  settings.decorated = true;
  settings.title = "Party Time";

  auto mainApp = make_shared<ofApp>();

  // Control window.
  auto controlWindow = ofCreateWindow(settings);
  controlWindow->setWindowPosition(ofGetScreenWidth() / 2 - settings.getWidth() / 2, ofGetScreenHeight() / 2 - settings.getHeight() / 2);

  // Main window.
  settings.setSize(1920, 1080);
  settings.setPosition(glm::vec2(1920, 0));
  settings.resizable = false;
  settings.decorated = false;
  settings.shareContextWith = controlWindow;
  auto projWindow = ofCreateWindow(settings);
  projWindow->setVerticalSync(false);
  ofAddListener(projWindow->events().draw, mainApp.get(), &ofApp::drawProjection);

  ofRunApp(controlWindow, mainApp);
  ofRunMainLoop();
}
```

One of the simplest ways to set this up is to use a second draw function in the same `ofApp`, which will draw in the second window. Note that by just redrawing our FBOs in the second window, we don't need to re-render the contours or the point cloud, as this is already saved in the frame buffer texture.

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

  void drawProjection(ofEventArgs& args);

  void keyPressed(int key);

  void deviceAdded(string& serialNumber);

  ofxRealSense2::Context rsContext;

  ofImage thresholdImg;

  ofxCv::ContourFinder contourFinder;
  
  ofEasyCam camera;

  ofFbo visionFbo;
  ofFbo pointsFbo;

  ofParameter<float> nearThreshold;
  ofParameter<float> farThreshold;

  ofParameter<float> minArea;
  ofParameter<float> maxArea;

  ofParameter<int> drawMode;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  // Texture coordinates from RealSense are normalized (between 0-1).
  // This call normalizes all OF texture coordinates so that they match.
  ofDisableArbTex();

  // Start the RealSense context.
  // Devices are added in the deviceAdded() callback function.
  ofAddListener(rsContext.deviceAddedEvent, this, &ofApp::deviceAdded);
  rsContext.setup(false);

  // Allocate the frame buffers.
  ofFboSettings settings;
  settings.width = 640;
  settings.height = 360;
  visionFbo.allocate(settings);
  pointsFbo.allocate(settings);

  // Setup the parameters.
  nearThreshold.set("Near Threshold", 0.01f, 0.0f, 0.1f);
  farThreshold.set("Far Threshold", 0.02f, 0.0f, 0.1f);
  minArea.set("Min Area", 0.01f, 0, 0.5f);
  maxArea.set("Max Area", 0.05f, 0, 0.5f);
  drawMode.set("Draw Mode", 0, 0, 3);

  // Setup the gui.
  guiPanel.setup("Depth Threshold", "settings.json");
  guiPanel.add(nearThreshold);
  guiPanel.add(farThreshold);
  guiPanel.add(minArea);
  guiPanel.add(maxArea);
  guiPanel.add(drawMode);
}

void ofApp::deviceAdded(string& serialNumber)
{
  ofLogNotice(__FUNCTION__) << "Starting device " << serialNumber;
  auto device = rsContext.getDevice(serialNumber);
  device->enableDepth();
  device->enableColor();
  device->enablePoints();
  device->startPipeline();

  // Uncomment this to add the device specific settings to the GUI.
  //guiPanel.add(device->params);
}

void ofApp::update()
{
  rsContext.update();

  shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice && rsDevice->isFrameNew())
  {
    // Threshold the depth.
    ofFloatPixels rawDepthPix = rsDevice->getRawDepthPix();
    ofFloatPixels thresholdNear, thresholdFar, thresholdResult;
    ofxCv::threshold(rawDepthPix, thresholdNear, nearThreshold);
    ofxCv::threshold(rawDepthPix, thresholdFar, farThreshold, true);
    ofxCv::bitwise_and(thresholdNear, thresholdFar, thresholdResult);
    thresholdImg.setFromPixels(thresholdResult);

    // Find contours.
    contourFinder.setMinAreaNorm(minArea);
    contourFinder.setMaxAreaNorm(maxArea);
    contourFinder.findContours(thresholdImg);

    // Draw CV operations.
    visionFbo.begin();
    {
      // Draw the threshold background.
      ofSetColor(255);
      thresholdImg.draw(0, 0);

      // Draw the contours over it.
      ofSetColor(0, 255, 0);
      contourFinder.draw();
    }
    visionFbo.end();

    // Draw 3D world.
    pointsFbo.begin();
    {
      ofBackground(0);

      camera.begin();
      ofEnableDepthTest();
      ofPushMatrix();

      // Adjust points to match OF 3D space.
      ofScale(1000);
      ofRotateXDeg(180);

      rsDevice->getColorTex().bind();
      rsDevice->getPointsMesh().draw();
      rsDevice->getColorTex().unbind();
      
      ofPopMatrix();
      ofDisableDepthTest();
      camera.end();
    }
    pointsFbo.end();
  }
}

void ofApp::draw()
{
  shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    // 2x2 window.
    int frameWidth = ofGetWidth() / 2;
    int frameHeight = ofGetHeight() / 2;

    rsDevice->getDepthTex().draw(0, 0, frameWidth, frameHeight);
    rsDevice->getColorTex().draw(0, frameHeight, frameWidth, frameHeight);
    visionFbo.draw(frameWidth, 0, frameWidth, frameHeight);

    ofRectangle camDrawRect = ofRectangle(frameWidth, frameHeight, frameWidth, frameHeight);
    pointsFbo.draw(camDrawRect);

    camera.setControlArea(camDrawRect);
  }

  // Draw the gui.
  guiPanel.draw();
}

void ofApp::drawProjection(ofEventArgs& args)
{
  shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    if (drawMode == 0)
    {
      // Draw fullscreen depth.
      rsDevice->getDepthTex().draw(ofGetCurrentViewport());
    }
    else if (drawMode == 1)
    {
      // Draw fullscreen color.
      rsDevice->getColorTex().draw(ofGetCurrentViewport());
    }
    else if (drawMode == 2)
    {
      // Draw fullscreen vision.
      visionFbo.draw(ofGetCurrentViewport());
    }
    else if (drawMode == 3)
    {
      // Draw fullscreen points.
      pointsFbo.draw(ofGetCurrentViewport());
    }
  }
}

void ofApp::keyPressed(int key)
{
  if (key == ' ')
  {
    drawMode = (drawMode + 1) % 4;
  }
}
```

## Blending

If we think of FBOs as layers, we can get interesting results by stacking them and applying different blend modes to them. OpenGL has built-in blend functions that are used to calculate the color of pixels that are overlaid in the same buffer, and these can be accessed using the [`ofEnableBlendMode()`](https://openframeworks.cc/documentation/graphics/ofGraphics/#!show_ofEnableBlendMode) global function. If you use Photoshop, some of these like `OF_BLENDMODE_ADD` and `OF_BLENDMODE_MULTIPLY` should sound familiar.

Let's update our RealSense example to render the point cloud in the projection window using custom blend modes.

* Set the RealSense alignment mode to `Align::Color` to ensure that depth, color, and point frames are in the same coordinate space.
* Use a new FBO to render a masked color image, by multiplying the threshold image by the full color image. This is done using `OF_BLENDMODE_MULTIPLY`. Note that this could have been achieved using OpenCV, but using blend modes happens almost automatically on the `ofTexture`!
* Draw the point cloud using the masked color image for texture, and `OF_BLENDMODE_SCREEN`. This ensures that color pixels override any black pixels and get drawn, even if they are behind the black pixels in 3D space.

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

  void drawProjection(ofEventArgs& args);

  void deviceAdded(string& serialNumber);

  ofxRealSense2::Context rsContext;

  ofImage thresholdImg;

  ofxCv::ContourFinder contourFinder;
  
  ofEasyCam camera;

  ofFbo visionFbo;
  ofFbo colorFbo;
  ofFbo pointsFbo;

  ofParameter<float> nearThreshold;
  ofParameter<float> farThreshold;

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
  // Texture coordinates from RealSense are normalized (between 0-1).
  // This call normalizes all OF texture coordinates so that they match.
  ofDisableArbTex();

  // Start the RealSense context.
  // Devices are added in the deviceAdded() callback function.
  ofAddListener(rsContext.deviceAddedEvent, this, &ofApp::deviceAdded);
  rsContext.setup(false);

  // Allocate the frame buffers.
  ofFboSettings settings;
  settings.width = 640;
  settings.height = 360;
  visionFbo.allocate(settings);
  colorFbo.allocate(settings);
  pointsFbo.allocate(settings);

  // Setup the parameters.
  nearThreshold.set("Near Threshold", 0.01f, 0.0f, 0.1f);
  farThreshold.set("Far Threshold", 0.02f, 0.0f, 0.1f);
  
  minArea.set("Min Area", 0.01f, 0, 0.5f);
  maxArea.set("Max Area", 0.05f, 0, 0.5f);

  // Setup the gui.
  guiPanel.setup("Depth Threshold", "settings.json");
  guiPanel.add(nearThreshold);
  guiPanel.add(farThreshold);
  guiPanel.add(minArea);
  guiPanel.add(maxArea);
}

void ofApp::deviceAdded(string& serialNumber)
{
  ofLogNotice(__FUNCTION__) << "Starting device " << serialNumber;
  auto device = rsContext.getDevice(serialNumber);
  device->enableDepth();
  device->enableColor();
  device->enablePoints();
  device->startPipeline();

  device->alignMode = ofxRealSense2::Device::Align::Color;

  // Uncomment this to add the device specific settings to the GUI.
  //guiPanel.add(device->params);
}

void ofApp::update()
{
  rsContext.update();

  shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice && rsDevice->isFrameNew())
  {
    // Threshold the depth.
    ofFloatPixels rawDepthPix = rsDevice->getRawDepthPix();
    ofFloatPixels thresholdNear, thresholdFar, thresholdResult;
    ofxCv::threshold(rawDepthPix, thresholdNear, nearThreshold);
    ofxCv::threshold(rawDepthPix, thresholdFar, farThreshold, true);
    ofxCv::bitwise_and(thresholdNear, thresholdFar, thresholdResult);
    thresholdImg.setFromPixels(thresholdResult);

    // Find contours.
    contourFinder.setMinAreaNorm(minArea);
    contourFinder.setMaxAreaNorm(maxArea);
    contourFinder.findContours(thresholdImg);

    // Draw CV operations.
    visionFbo.begin();
    {
      // Draw the threshold background.
      ofSetColor(255);
      thresholdImg.draw(0, 0);

      // Draw the contours over it.
      ofSetColor(0, 255, 0);
      contourFinder.draw();
    }
    visionFbo.end();

    // Draw masked color.
    colorFbo.begin();
    {
      // Draw the color background.
      rsDevice->getColorTex().draw(0, 0);

      // Set the blend mode to multiply to clip out any black pixels.
      ofEnableBlendMode(OF_BLENDMODE_MULTIPLY);

      // Draw the threshold image on top.
      // The result will be a colored threshold image.
      thresholdImg.draw(0, 0);
    }
    colorFbo.end();

    // Draw 3D world.
    pointsFbo.begin();
    {
      ofBackground(0);

      camera.begin();
      ofPushMatrix();

      // Adjust points to match OF 3D space.
      ofScale(1000);
      ofRotateXDeg(180);

      // Set the blend mode to screen to let any pixels with color through.
      ofEnableBlendMode(OF_BLENDMODE_SCREEN);

      // Draw the point cloud using the masked color texture.
      colorFbo.getTexture().bind();
      rsDevice->getPointsMesh().draw();
      colorFbo.getTexture().unbind();
      
      ofPopMatrix();
      camera.end();
    }
    pointsFbo.end();
  }
}

void ofApp::draw()
{
  shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    // 2x2 window.
    int frameWidth = ofGetWidth() / 2;
    int frameHeight = ofGetHeight() / 2;

    rsDevice->getDepthTex().draw(0, 0, frameWidth, frameHeight);
    colorFbo.draw(0, frameHeight, frameWidth, frameHeight);
    visionFbo.draw(frameWidth, 0, frameWidth, frameHeight);

    ofRectangle camDrawRect = ofRectangle(frameWidth, frameHeight, frameWidth, frameHeight);
    pointsFbo.draw(camDrawRect);

    camera.setControlArea(camDrawRect);
  }

  // Draw the gui.
  guiPanel.draw();
}

void ofApp::drawProjection(ofEventArgs& args)
{
  // Draw fullscreen points.
  pointsFbo.draw(ofGetCurrentViewport());
}
```

Finally, we can add custom alpha blending on the projection output.

* Change the background in the points FBO from `ofBackground(0)` to [`ofClear(0, 0)`](https://openframeworks.cc/documentation/graphics/ofGraphics/#!show_ofClear). `ofClear()` works like `ofBackground()` but can also take a second parameter for alpha. By clearing alpha to `0`, we are making the FBO texture transparent, which will make more interesting overlays.
* Add a call to [`ofSetBackgroundAuto(false)`](https://openframeworks.cc/documentation/graphics/ofGraphics/#show_ofSetBackgroundAuto) at the start of `drawProjection()`. This disables auto-clearing the window and will allow us to overlay textures over many frames. Note that we usually do not need to call `ofSetBackgroundAuto()` every frame, but since we are setting it for our second window, this is the best place to call this function.
* Draw the background color and the point cloud FBO using a variable alpha value to get different ghostly effects.

{{< image src="fbo-fade.png" alt="FBO Fade" caption="*FBO Fade*" width="600px" >}}

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

  void drawProjection(ofEventArgs& args);

  void deviceAdded(string& serialNumber);

  ofxRealSense2::Context rsContext;

  ofImage thresholdImg;

  ofxCv::ContourFinder contourFinder;
  
  ofEasyCam camera;

  ofFbo visionFbo;
  ofFbo colorFbo;
  ofFbo pointsFbo;

  ofParameter<float> nearThreshold;
  ofParameter<float> farThreshold;

  ofParameter<float> minArea;
  ofParameter<float> maxArea;

  ofParameter<int> fadeAlpha;
  ofParameter<int> drawAlpha;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  // Texture coordinates from RealSense are normalized (between 0-1).
  // This call normalizes all OF texture coordinates so that they match.
  ofDisableArbTex();

  // Start the RealSense context.
  // Devices are added in the deviceAdded() callback function.
  ofAddListener(rsContext.deviceAddedEvent, this, &ofApp::deviceAdded);
  rsContext.setup(false);

  // Allocate the frame buffers.
  ofFboSettings settings;
  settings.width = 640;
  settings.height = 360;
  visionFbo.allocate(settings);
  colorFbo.allocate(settings);
  pointsFbo.allocate(settings);

  // Setup the parameters.
  nearThreshold.set("Near Threshold", 0.01f, 0.0f, 0.1f);
  farThreshold.set("Far Threshold", 0.02f, 0.0f, 0.1f);
  
  minArea.set("Min Area", 0.01f, 0, 0.5f);
  maxArea.set("Max Area", 0.05f, 0, 0.5f);

  fadeAlpha.set("Fade Alpha", 16, 0, 255);
  drawAlpha.set("Draw Alpha", 127, 0, 255);

  // Setup the gui.
  guiPanel.setup("Depth Threshold", "settings.json");
  guiPanel.add(nearThreshold);
  guiPanel.add(farThreshold);
  guiPanel.add(minArea);
  guiPanel.add(maxArea);
  guiPanel.add(fadeAlpha);
  guiPanel.add(drawAlpha);
}

void ofApp::deviceAdded(std::string& serialNumber)
{
  ofLogNotice(__FUNCTION__) << "Starting device " << serialNumber;
  auto device = rsContext.getDevice(serialNumber);
  device->enableDepth();
  device->enableColor();
  device->enablePoints();
  device->startPipeline();

  device->alignMode = ofxRealSense2::Device::Align::Color;

  // Uncomment this to add the device specific settings to the GUI.
  //guiPanel.add(device->params);
}

void ofApp::update()
{
  rsContext.update();

  shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice && rsDevice->isFrameNew())
  {
    // Threshold the depth.
    ofFloatPixels rawDepthPix = rsDevice->getRawDepthPix();
    ofFloatPixels thresholdNear, thresholdFar, thresholdResult;
    ofxCv::threshold(rawDepthPix, thresholdNear, nearThreshold);
    ofxCv::threshold(rawDepthPix, thresholdFar, farThreshold, true);
    ofxCv::bitwise_and(thresholdNear, thresholdFar, thresholdResult);
    thresholdImg.setFromPixels(thresholdResult);

    // Find contours.
    contourFinder.setMinAreaNorm(minArea);
    contourFinder.setMaxAreaNorm(maxArea);
    contourFinder.findContours(thresholdImg);

    // Draw CV operations.
    visionFbo.begin();
    {
      // Draw the threshold background.
      ofSetColor(255);
      thresholdImg.draw(0, 0);

      // Draw the contours over it.
      ofSetColor(0, 255, 0);
      contourFinder.draw();
    }
    visionFbo.end();

    // Draw masked color.
    colorFbo.begin();
    {
      // Draw the color background.
      rsDevice->getColorTex().draw(0, 0);

      // Set the blend mode to multiply to clip out any black pixels.
      ofEnableBlendMode(OF_BLENDMODE_MULTIPLY);

      // Draw the threshold image on top.
      // The result will be a colored threshold image.
      thresholdImg.draw(0, 0);
    }
    colorFbo.end();

    // Draw 3D world.
    pointsFbo.begin();
    {
      ofClear(0, 0);

      camera.begin();
      ofPushMatrix();

      // Adjust points to match OF 3D space.
      ofScale(1000);
      ofRotateXDeg(180);

      // Set the blend mode to screen to let any pixels with color through.
      ofEnableBlendMode(OF_BLENDMODE_SCREEN);

      // Draw the point cloud using the masked color texture.
      colorFbo.getTexture().bind();
      rsDevice->getPointsMesh().draw();
      colorFbo.getTexture().unbind();
      
      ofPopMatrix();
      camera.end();
    }
    pointsFbo.end();
  }
}

void ofApp::draw()
{
  shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    // 2x2 window.
    int frameWidth = ofGetWidth() / 2;
    int frameHeight = ofGetHeight() / 2;

    rsDevice->getDepthTex().draw(0, 0, frameWidth, frameHeight);
    colorFbo.draw(0, frameHeight, frameWidth, frameHeight);
    visionFbo.draw(frameWidth, 0, frameWidth, frameHeight);

    ofRectangle camDrawRect = ofRectangle(frameWidth, frameHeight, frameWidth, frameHeight);
    pointsFbo.draw(camDrawRect);

    camera.setControlArea(camDrawRect);
  }

  // Draw the gui.
  guiPanel.draw();
}

void ofApp::drawProjection(ofEventArgs& args)
{
  // Disable clearing the background automatically.
  ofSetBackgroundAuto(false);

  // Draw a background color with optional transparency.
  ofSetColor(0, fadeAlpha);
  ofDrawRectangle(ofGetCurrentViewport());

  // Draw fullscreen points.
  ofSetColor(255, drawAlpha);
  pointsFbo.draw(ofGetCurrentViewport());
}
```

<figure style="width:600px;height:400px;display:block;margin:0 auto;">
  <iframe width="600" height="375" src="https://www.youtube.com/embed/haX3AIHSQ1E" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
  <figcaption><i>rag & bone elevates the catwalk with technology</i></figcaption>
</figure>
