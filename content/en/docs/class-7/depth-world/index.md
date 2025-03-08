---
title: "Depth World"
description: ""
lead: ""
date: 2022-10-27T19:16:21-04:00
lastmod: 2022-10-27T19:16:21-04:00
draft: false
images: []
menu:
  docs:
    parent: "class-7"
    identifier: "depth-world"
weight: 720
toc: true
---

## Point Clouds

If we change our mesh topology to `OF_PRIMITIVE_POINTS`, this will draw points at each vertex position without interpolating in the space between them.

If we want to add more resolution, we need more points.

{{< details "Update the mesh to render points, and add a vertex for every pixel." >}}

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

  ofVideoGrabber grabber;

  ofMesh pointMesh;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  // Start the grabber.
  grabber.setup(640, 480);

  pointMesh.setMode(OF_PRIMITIVE_POINTS);
  for (int y = 0; y < grabber.getHeight(); y++)
  {
    for (int x = 0; x < grabber.getWidth(); x++)
    {
      pointMesh.addVertex(glm::vec3(x, y, 0));
      pointMesh.addTexCoord(glm::vec2(x, y));
    }
  }

  // Setup the gui.
  guiPanel.setup("3D World", "settings.json");
}

void ofApp::update()
{
  grabber.update();
}

void ofApp::draw()
{
  // Render the mesh.
  grabber.bind();
  pointMesh.draw();
  grabber.unbind();

  // Draw the gui.
  guiPanel.draw();
}
```

This still looks almost identical because we're drawing a point at every pixel position, essentially leaving no gaps.

{{< /details >}}

{{< details "Add a skip parameter to modify the distance between points." >}}

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

  ofVideoGrabber grabber;

  ofMesh pointMesh;

  ofParameter<int> skipPoints;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  // Start the grabber.
  grabber.setup(640, 480);

  // Setup the parameters.
  skipPoints.set("Skip Points", 1, 1, 24);

  // Setup the gui.
  guiPanel.setup("3D World", "settings.json");
  guiPanel.add(skipPoints);
}

void ofApp::update()
{
  grabber.update();

  // Rebuild the mesh.
  pointMesh.clear();
  pointMesh.setMode(OF_PRIMITIVE_POINTS);
  for (int y = 0; y < grabber.getHeight(); y += skipPoints)
  {
    for (int x = 0; x < grabber.getWidth(); x += skipPoints)
    {
      pointMesh.addVertex(glm::vec3(x, y, 0));
      pointMesh.addTexCoord(glm::vec2(x, y));
    }
  }
}

void ofApp::draw()
{
  // Render the mesh.
  grabber.bind();
  pointMesh.draw();
  grabber.unbind();

  // Draw the gui.
  guiPanel.draw();
}
```

{{< /details >}}

Instead of setting the `z` position of each vertex to `0`, we can use some meaningful data. Let's switch input from `ofVideoGrabber` to a depth camera, and use the depth data from our sensor!

{{< image src="rs-pointsgray.png" alt="RealSense Points Gray" caption="*RealSense Points Gray*" width="600px" >}}

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

  ofMesh pointMesh;

  ofParameter<int> skipPoints;

  ofEasyCam cam;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 360);

  // Start the depth sensor.
  rsContext.setup(true);

  // Setup the parameters.
  skipPoints.set("Skip Points", 1, 1, 24);

  // Setup the gui.
  guiPanel.setup("3D World", "settings.json");
  guiPanel.add(skipPoints);
}

void ofApp::update()
{
  rsContext.update();

  std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    ofShortPixels rawDepthPix = rsDevice->getRawDepthPix();

    // Rebuild the mesh.
    pointMesh.clear();
    pointMesh.setMode(OF_PRIMITIVE_POINTS);
    for (int y = 0; y < rsDevice->getRawDepthTex().getHeight(); y += skipPoints)
    {
      for (int x = 0; x < rsDevice->getRawDepthTex().getWidth(); x += skipPoints)
      {
        int depth = rawDepthPix.getColor(x, y).r;
        pointMesh.addVertex(glm::vec3(x, y, depth));
        pointMesh.addTexCoord(glm::vec2(x, y));
      }
    }
  }
}

void ofApp::draw()
{
  // Try to get a pointer to a device.
  std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);

  // Begin rendering through the camera.
  cam.begin();
  ofEnableDepthTest();

  if (rsDevice)
  {
    // Render the mesh.
    rsDevice->getDepthTex().bind();
    pointMesh.draw();
    rsDevice->getDepthTex().unbind();
  }

  ofDisableDepthTest();
  cam.end();
  // Done rendering through the camera.

  // Draw the gui.
  guiPanel.draw();
}
```

Note the use of [`ofEnableDepthTest()`](https://openframeworks.cc/documentation/graphics/ofGraphics/#!show_ofEnableDepthTest) inside the camera block. This is to make sure that our points' distance from the camera is taken into account when rendering.

* With depth test on, geometry is sorted before rendering so that points that are nearer than others will be drawn on top of them.
* When depth test is off (which is how we've been drawing so far), geometry is sorted in the order it is drawn to the screen, without taking its position in space into account.

<figure style="width:600px;height:360px;display:block;margin:0 auto;">
<iframe src="https://player.vimeo.com/video/367810259" width="600" height="337" frameborder="0" allow="autoplay; fullscreen; picture-in-picture" allowfullscreen></iframe>
<figcaption><i><a href="https://vimeo.com/367810259">rag & bone, Drums & Bots, BTS</a> from <a href="https://vimeo.com/specialguestco">SpecialGuest</a> on <a href="https://vimeo.com">Vimeo</a>.</i></figcaption>
</figure>

## Correspondence

At this point, we may be tempted to swap out the bind from the depth texture to the color texture, to get points in the correct color. If we do so we will notice the results don't quite match up.

{{< image src="rs-pointsbad.png" alt="RealSense Points Bad" caption="*RealSense Points Bad*" width="600px" >}}

The depth cameras we are using are actually made up of a couple of sensors: an infrared sensor to record depth and a color sensor to record... color. These are two different components, each with their own lens, resolution, and field of view. Because of this, pixels at the same coordinates will most likely not match. There needs to be some type of formula to convert from depth space to color space and vice-versa. Moreover, the world positions are also in their own space. Pixel coordinate `(120, 12)` doesn't mean anything in 3D, so there needs to be another formula to convert from depth space to world space, and from color space to world space.

Luckily for us, depth cameras have all this already figured out and provide this information through their SDK.

This can have many names, but it's usually called *registration*, *alignment*, or *correspondence*.

There are a few options in how this is delivered.

* Secondary textures where the pixels are transformed to make a 1-1 relationship. For example, `ofxKinect` has a [`setRegistration()`](https://openframeworks.cc/documentation/ofxKinect/ofxKinect/#!show_setRegistration) function which outputs the color data in the same space as the depth data.
* Look-up tables (LUTs) (usually as textures) where you can calculate a pixel's world position based on the pixel value in the LUT. For example, [`ofxKinectForWindows2`](https://github.com/elliotwoods/ofxKinectForWindows2) uses an RGB float texture can be used to represent a vertex XYZ position.
* A getter function to map a pixel coordinate to a world position. For example, [`ofxKinectV2`](https://github.com/ofTheo/ofxKinectV2) has a `getWorldCoordinateAt(x, y)` function that does exactly that.
* A pre-mapped mesh that already has the depth and color mapped to world positions and provides the output data. For example, [`ofxRealSense2`](https://gitlab.com/prisonerjohn/ofxrealsense2/) has a `getPointsMesh()` function that can be drawn directly.

You will have to read the documentation or look at the examples for the depth camera you are using to determine how to get correspondence between depth, color, and world space.

### Intel RealSense

For our app, we can set the `ofxRealSense2::Device::alignMode` parameter to `Align::Color` to align the depth and color frames.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 360);

  // Start the depth sensor.
  rsContext.setup(true);

  // Setup the parameters.
  skipPoints.set("Skip Points", 1, 1, 24);

  // Setup the gui.
  guiPanel.setup("3D World", "settings.json");
  guiPanel.add(skipPoints);
}

void ofApp::update()
{
  rsContext.update();

  std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    // Align the frames to the color viewport.
    rsDevice->alignMode = ofxRealSense2::Device::Align::Color;

    ofShortPixels rawDepthPix = rsDevice->getRawDepthPix();

    // Rebuild the mesh.
    pointMesh.clear();
    pointMesh.setMode(OF_PRIMITIVE_POINTS);
    for (int y = 0; y < rsDevice->getRawDepthTex().getHeight(); y += skipPoints)
    {
      for (int x = 0; x < rsDevice->getRawDepthTex().getWidth(); x += skipPoints)
      {
        int depth = rawDepthPix.getColor(x, y).r;
        pointMesh.addVertex(ofDefaultVec3(x, y, depth));
        pointMesh.addTexCoord(ofDefaultVec2(x, y));
      }
    }
  }
}

void ofApp::draw()
{
  // Try to get a pointer to a device.
  std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);

  // Begin rendering through the camera.
  cam.begin();
  ofEnableDepthTest();

  if (rsDevice)
  {
    // Render the mesh.
    rsDevice->getColorTex().bind();
    pointMesh.draw();
    rsDevice->getColorTex().unbind();
  }

  ofDisableDepthTest();
  cam.end();
  // Done rendering through the camera.

  // Draw the gui.
  guiPanel.draw();
}
```

Note that this still doesn't place our points in world space but at least they are colored accurately. To do this, we can use the `ofxRealSense2::Device::getWorldPosition(x, y)` function.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 360);

  // Start the depth sensor.
  rsContext.setup(true);

  // Setup the parameters.
  skipPoints.set("Skip Points", 1, 1, 24);

  // Setup the gui.
  guiPanel.setup("3D World", "settings.json");
  guiPanel.add(skipPoints);
}

void ofApp::update()
{
  rsContext.update();

  std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
  if (rsDevice)
  {
    // Align the frames to the color viewport.
    rsDevice->alignMode = ofxRealSense2::Device::Align::Color;

    // Enable world point calculation.
    rsDevice->enablePoints();

    ofShortPixels rawDepthPix = rsDevice->getRawDepthPix();

    // Rebuild the mesh.
    pointMesh.clear();
    pointMesh.setMode(OF_PRIMITIVE_POINTS);
    for (int y = 0; y < rsDevice->getRawDepthTex().getHeight(); y += skipPoints)
    {
      for (int x = 0; x < rsDevice->getRawDepthTex().getWidth(); x += skipPoints)
      {
        pointMesh.addVertex(rsDevice->getWorldPosition(x, y));
        pointMesh.addTexCoord(glm::vec2(x, y));
      }
    }
  }
}

void ofApp::draw()
{
  // Try to get a pointer to a device.
  std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);

  // Begin rendering through the camera.
  cam.begin();
  ofEnableDepthTest();
  ofScale(100);

  if (rsDevice)
  {
    // Render the mesh.
    rsDevice->getColorTex().bind();
    pointMesh.draw();
    rsDevice->getColorTex().unbind();
  }

  ofDisableDepthTest();
  cam.end();
  // Done rendering through the camera.

  // Draw the gui.
  guiPanel.draw();
}
```

### Microsoft Kinect

For `ofxKinect`, we just need to call `setRegistration()` to have the depth and color images correspond.

Here is the extruded texture example.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"
#include "ofxKinect.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  ofxKinect kinect;

  ofMesh pointMesh;

  ofParameter<int> skipPoints;

  ofEasyCam cam;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  // Start the depth sensor.
  kinect.setRegistration(true);
  kinect.init();
  kinect.open();

  // Setup the parameters.
  skipPoints.set("Skip Points", 1, 1, 24);

  // Setup the gui.
  guiPanel.setup("3D World", "settings.json");
  guiPanel.add(skipPoints);
}

void ofApp::update()
{
  kinect.update();

  if (kinect.isFrameNew())
  {
    ofShortPixels rawDepthPix = kinect.getRawDepthPixels();

    // Rebuild the mesh.
    pointMesh.clear();
    pointMesh.setMode(OF_PRIMITIVE_POINTS);
    for (int y = 0; y < kinect.getHeight(); y += skipPoints)
    {
      for (int x = 0; x < kinect.getWidth(); x += skipPoints)
      {
        int depth = rawDepthPix.getColor(x, y).r;
        pointMesh.addVertex(glm::vec3(x, y, depth));
        pointMesh.addTexCoord(glm::vec2(x, y));
      }
    }
  }
}

void ofApp::draw()
{
  ofBackground(0);

  // Begin rendering through the camera.
  cam.begin();
  ofEnableDepthTest();

  // Render the mesh.
  kinect.getTexture().bind();
  pointMesh.draw();
  kinect.getTexture().unbind();

  ofDisableDepthTest();
  cam.end();
  // Done rendering through the camera.

  // Draw the gui.
  guiPanel.draw();
}
```

And here is the example adjusted for world positions, using [`ofxKinect.getWorldCooordinateAt()`](https://openframeworks.cc//documentation/ofxKinect/ofxKinect/#!show_getWorldCoordinateAt).

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  // Start the depth sensor.
  kinect.setRegistration(true);
  kinect.init();
  kinect.open();

  // Setup the parameters.
  skipPoints.set("Skip Points", 1, 1, 24);

  // Setup the gui.
  guiPanel.setup("3D World", "settings.json");
  guiPanel.add(skipPoints);
}

void ofApp::update()
{
  kinect.update();

  if (kinect.isFrameNew())
  {
    // Rebuild the mesh.
    pointMesh.clear();
    pointMesh.setMode(OF_PRIMITIVE_POINTS);
    for (int y = 0; y < kinect.getHeight(); y += skipPoints)
    {
      for (int x = 0; x < kinect.getWidth(); x += skipPoints)
      {
        pointMesh.addVertex(kinect.getWorldCoordinateAt(x, y));
        pointMesh.addTexCoord(glm::vec2(x, y));
      }
    }
  }
}

void ofApp::draw()
{
  ofBackground(0);

  // Begin rendering through the camera.
  cam.begin();
  ofEnableDepthTest();

  // Render the mesh.
  kinect.getTexture().bind();
  pointMesh.draw();
  kinect.getTexture().unbind();

  ofDisableDepthTest();
  cam.end();
  // Done rendering through the camera.

  // Draw the gui.
  guiPanel.draw();
}
```

<figure style="width:600px;height:400px;display:block;margin:0 auto;">
<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/89680830?color=ffffff&title=0&byline=0&portrait=0" style="position:absolute;top:0;left:0;width:100%;height:100%;" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>
<figcaption><i><a href="https://vimeo.com/89680830">CLOUDS - Overview</a> from <a href="https://vimeo.com/deepspeed">Jonathan Minard</a> on <a href="https://vimeo.com">Vimeo</a>.</i></figcaption>
</figure>
