---
title: "Depth World"
description: ""
lead: ""
date: 2022-10-27T19:16:21-04:00
lastmod: 2022-10-27T19:16:21-04:00
draft: true
images: []
menu:
  docs:
    parent: "class-7"
    identifier: "depth-world"
weight: 710
toc: true
---

## 3D in OF

Everything we have done so far has been in two dimensions, using `(x, y)` for position, and `[width, height]` for size. Now that we've got data that also has depth, we can move into the third dimension to represent it. This means using `(x, y, z)` for position, and `[width, height, depth]` for size.

By default, the openFrameworks canvas is set to 2D. The origin `(0, 0)` is in the top-left, and values increase as we move right and down. In 3D, the origin `(0, 0, 0)` is in the middle of the window. We can move in both directions in any of the three dimensions, meaning that we can have positive and negative values for positions.

Note that in 3D, the Y direction is the opposite than in 2D; Y increases as we move up. This is common with most 3D software.

The Z direction, however, can follow one of two coordinate system conventions:

* A *right-handed* coordinate system means the z-axis points forward. Z decreases as we move further away.
* A *left-handed* coordinate system means the z-axis points backwards. Z increases as we move further away.

openFrameworks follows the OpenGL convention and has a right-handed coordinate system.

{{< image src="axis.png" alt="OpenGL Axis" caption="*OpenGL Axis*" width="600px" >}}

## Cameras

The simplest way to switch to 3D space is to use an [`ofCamera`](https://openframeworks.cc/documentation/3d/ofCamera/).

* `ofCamera` has [`begin()`](https://openframeworks.cc/documentation/3d/ofCamera/#show_begin) and [`end()`](https://openframeworks.cc/documentation/3d/ofCamera/#show_end) functions; anything we put in between those will be drawn in 3D space.
* The camera needs to be positioned somewhere in space. We use [`setPosition()`](https://openframeworks.cc/documentation/3d/ofNode/#show_setPosition) to place it.

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

  ofCamera cam;

  ofParameter<glm::vec3> camPosition;

  ofParameter<bool> useCamera;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  grabber.setup(640, 480);

  // Setup the parameters.
  useCamera.set("Use Camera", false);
  camPosition.set("Cam Position", glm::vec3(0, 0, 90), glm::vec3(-100), glm::vec3(100));

  // Setup the gui.
  guiPanel.setup("3D World", "settings.json");
  guiPanel.add(useCamera);
  guiPanel.add(camPosition);
}

void ofApp::update()
{
  grabber.update();

  cam.setPosition(camPosition);
}

void ofApp::draw()
{
  if (useCamera)
  {
    // Begin rendering through the camera.
    cam.begin();

    // Scale the drawing down into more manageable units.
    ofScale(0.1f);
    // Draw the grabber image anchored in the center.
    grabber.draw(-grabber.getWidth() / 2, -grabber.getHeight() / 2);

    cam.end();
    // Done rendering through the camera.
  }
  else
  {
    // Draw the grabber in 2D.
    grabber.draw(0, 0);
  }

  // Draw the gui.
  guiPanel.draw();
}
```

Note the use of `glm::vec3` for the camera position attribute. [`glm`](https://openframeworks.cc/documentation/glm/) is the mathematics library used by default in OF. Here we are using the `glm::vec3` type which is a 3D point (with x, y, z coordinates).

* Alternatively to `setPosition()`, there are also functions for [`truck()`](https://openframeworks.cc/documentation/3d/ofNode/#show_truck), [`boom()`](https://openframeworks.cc/documentation/3d/ofNode/#show_boom) (aka pedestal), and [`dolly()`](https://openframeworks.cc/documentation/3d/ofNode/#show_dolly) to move in the XYZ axes.
* A camera can be oriented anywhere in space as well. We can set its orientation with [`setOrientation()`](https://openframeworks.cc/documentation/3d/ofNode/#show_setOrientation).
* There are also functions for [`panDeg()`](https://openframeworks.cc/documentation/3d/ofNode/#show_panDeg), [`tiltDeg()`](https://openframeworks.cc/documentation/3d/ofNode/#show_tiltDeg) (aka pedestal), and [`rollDeg()`](https://openframeworks.cc/documentation/3d/ofNode/#show_rollDeg) to rotate in the XYZ axes.

{{< image src="https://external-preview.redd.it/dgHPuSpFv8RvvFbQZW4zWCW3AAwTuW7yZog8g22F9bo.png?auto=webp&s=2acd6490a7e5b188871b80f52e678ded349d0811" alt="The names of types of camera movements" caption="[*The names of types of camera movements*](https://www.reddit.com/r/LearnUselessTalents/comments/5jpd54/the_names_of_types_of_camera_movements/)" width="600px" >}}

* It is sometimes more useful to tell a camera where to "look" rather than how to orient itself. This is done with [`lookAt()`](https://openframeworks.cc/documentation/3d/ofNode/#show_lookAt) which takes a target position as an argument. The camera will figure out automatically what orientation to use to look at that target.

The following example demonstrates the effect of these different functions on the `ofCamera`.

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

  ofCamera cam;

  ofParameter<glm::vec3> camPosition;
  ofParameter<float> camTruck;
  ofParameter<float> camBoom;
  ofParameter<float> camDolly;
  ofParameter<bool> orientCamera;
  ofParameter<glm::vec3> camLookAt;
  ofParameter<float> camPan;
  ofParameter<float> camTilt;
  ofParameter<float> camRoll;

  ofParameter<bool> useCamera;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  grabber.setup(640, 480);

  // Setup the parameters.
  useCamera.set("Use Camera", false);
  camPosition.set("Cam Position", glm::vec3(0, 0, 90), glm::vec3(-100), glm::vec3(100));
  camTruck.set("Truck", 0.0f, -100.0f, 100.0f);
  camBoom.set("Boom", 0.0f, -100.0f, 100.0f);
  camDolly.set("Dolly", 0.0f, -100.0f, 100.0f);
  orientCamera.set("Orient Camera", true);
  camLookAt.set("Cam Look At", ofDefaultVec3(0, 0, 0), ofDefaultVec3(-100), ofDefaultVec3(100));
  camPan.set("Pan", 0.0f, -90.0f, 90.0f);
  camTilt.set("Tilt", 0.0f, -90.0f, 90.0f);
  camRoll.set("Roll", 0.0f, -90.0f, 90.0f);

  // Setup the gui.
  guiPanel.setup("3D World", "settings.json");
  guiPanel.add(useCamera);
  guiPanel.add(camPosition);
  guiPanel.add(camTruck);
  guiPanel.add(camBoom);
  guiPanel.add(camDolly);
  guiPanel.add(orientCamera);
  guiPanel.add(camLookAt);
  guiPanel.add(camPan);
  guiPanel.add(camTilt);
  guiPanel.add(camRoll);
}

void ofApp::update()
{
  grabber.update();

  // Reset everything each frame, otherwise the transform will be additive.
  cam.resetTransform();

  cam.setPosition(camPosition);
  cam.truck(camTruck);
  cam.boom(camBoom);
  cam.dolly(camDolly);

  if (orientCamera)
  {
    cam.lookAt(camLookAt);
    cam.panDeg(camPan);
    cam.tiltDeg(camTilt);
    cam.rollDeg(camRoll);
  }
}

void ofApp::draw()
{
  if (useCamera)
  {
    // Begin rendering through the camera.
    cam.begin();

    // Scale the drawing down into more manageable units.
    ofScale(0.1f);
    // Draw the grabber image anchored in the center.
    grabber.draw(-grabber.getWidth() / 2, -grabber.getHeight() / 2);

    cam.end();
    // Done rendering through the camera.
  }
  else
  {
    // Draw the grabber in 2D.
    grabber.draw(0, 0);
  }

  // Draw the gui.
  guiPanel.draw();
}
```

Controlling an `ofCamera` using sliders is not always intuitive, so OF provides [`ofEasyCam`](https://openframeworks.cc/documentation/3d/ofEasyCam/) as an alternative. `ofEasyCam` is an `ofCamera` that is controlled using the mouse, and makes navigating the scene extremely easy. If you have every used 3D software like Maya or played FPS video games, controlling the camera should feel familiar.

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

  ofEasyCam cam;

  ofParameter<bool> useCamera;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  grabber.setup(640, 480);

  // Setup the parameters.
  useCamera.set("Use Camera", false);

  // Setup the gui.
  guiPanel.setup("3D World", "settings.json");
  guiPanel.add(useCamera);
}

void ofApp::update()
{
  grabber.update();
}

void ofApp::draw()
{
  if (useCamera)
  {
    // Begin rendering through the camera.
    cam.begin();

    // Scale the drawing down into more manageable units.
    ofScale(0.1f);
    // Draw the grabber image anchored in the center.
    grabber.draw(-grabber.getWidth() / 2, -grabber.getHeight() / 2);

    cam.end();
    // Done rendering through the camera.
  }
  else
  {
    // Draw the grabber in 2D.
    grabber.draw(0, 0);
  }

  // Draw the gui.
  guiPanel.draw();
}
```

## Meshes

Everything that is drawn on screen is *geometry*.

* The geometry has a *topology*, which is what the geometry is made up of. This is usually *points*, *lines*, or *triangles*. These can have different arrangements, as seen in the diagram below.
* The topology is made up of *vertices*. These are points with specific parameters. A simple *vertex* will just have a position. A more complex *vertex* can also have a color, a normal, texture coordinates, etc.
* The *topology* and *vertex* data together make up a *mesh*. A *mesh* is like a series of commands that tell the application how to draw the geometry to the screen.

In openFrameworks, we use [`ofMesh`](https://openframeworks.cc/documentation/3d/ofMesh/) as the geometry container. This is what is used to draw meshes to the screen.

{{< image src="https://personal.ntu.edu.sg/ehchua/programming/opengl/images/GL_GeometricPrimitives.png" alt="3D Graphics with OpenGL" caption="[*3D Graphics with OpenGL*](https://personal.ntu.edu.sg/ehchua/programming/opengl/CG_BasicsTheory.html)" width="600px" >}}

Every time we have been drawing images, we've actually been drawing two triangles in the shape of a rectangle.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void draw();

  ofMesh quadMesh;

  ofParameter<bool> drawWireframe;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  // Build the quad mesh.
  quadMesh.setMode(OF_PRIMITIVE_TRIANGLES);

  quadMesh.addVertex(glm::vec3(0, 0, 0));
  quadMesh.addVertex(glm::vec3(640, 0, 0));
  quadMesh.addVertex(glm::vec3(640, 480, 0));
  quadMesh.addVertex(glm::vec3(0, 0, 0));
  quadMesh.addVertex(glm::vec3(640, 480, 0));
  quadMesh.addVertex(glm::vec3(0, 480, 0));

  // Setup the parameters.
  drawWireframe.set("Wireframe?", false);

  // Setup the gui.
  guiPanel.setup("3D World", "settings.json");
  guiPanel.add(drawWireframe);
}

void ofApp::draw()
{
  // Render the mesh.
  if (drawWireframe)
  {
    quadMesh.drawWireframe();
  }
  else
  {
    quadMesh.draw();
  }

  // Draw the gui.
  guiPanel.draw();
}
```

{{< image src="quad-wires.png" alt="Quad Wires" caption="*Quad Wires*" width="600px" >}}

If we assign each vertex a color, we can see how that gets rendered across the geometry.

{{< details "Use <a href=\"https://openframeworks.cc/documentation/3d/ofMesh/#show_addColor\"><code>ofMesh::addColor()</code></a> to add a color attribute to each vertex in our quad mesh." >}}

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  // Build the quad mesh.
  quadMesh.setMode(OF_PRIMITIVE_TRIANGLES);
  
  quadMesh.addVertex(glm::vec3(0, 0, 0));
  quadMesh.addVertex(glm::vec3(640, 0, 0));
  quadMesh.addVertex(glm::vec3(640, 480, 0));
  quadMesh.addVertex(glm::vec3(0, 0, 0));
  quadMesh.addVertex(glm::vec3(640, 480, 0));
  quadMesh.addVertex(glm::vec3(0, 480, 0));

  quadMesh.addColor(ofColor(200, 0, 0));
  quadMesh.addColor(ofColor(0, 200, 0));
  quadMesh.addColor(ofColor(0, 0, 200));
  quadMesh.addColor(ofColor(200, 0, 0));
  quadMesh.addColor(ofColor(0, 0, 200));
  quadMesh.addColor(ofColor(200, 200, 0));

  // Setup the parameters.
  drawWireframe.set("Wireframe?", false);

  // Setup the gui.
  guiPanel.setup("3D World", "settings.json");
  guiPanel.add(drawWireframe);
}

void ofApp::draw()
{
  // Render the mesh.
  if (drawWireframe)
  {
    quadMesh.drawWireframe();
  }
  else
  {
    quadMesh.draw();
  }

  // Draw the gui.
  guiPanel.draw();
}
```

{{< /details >}}

Even though we only have four vertices and four colors, the entire window has color across it. When using topology `OF_PRIMITIVE_TRIANGLES`, the entire shape of the triangle is filled in, so the renderer calculates the color for each pixel on screen by mixing the vertex colors together. This is called *interpolation*.

{{< image src="quad-colors.png" alt="Quad Colors" caption="*Quad Colors*" width="600px" >}}

Interpolation doesn't just work with colors, but with all vertex parameters. This is why when we use two points to draw a line, the entire length of the line gets drawn and not just the two end points.

Let's replace the colors per vertex by texture coordinates. Instead of setting the color explicitly, this tells the renderer to pick the color from a texture we assign to it.

* This is called texture *binding*.
* Texture coordinates are 2D, so we will use `glm::vec2` to define them.
* All objects in OF that render to the screen have [`bind()`](https://openframeworks.cc/documentation/gl/ofTexture/#show_bind) / [`unbind()`](https://openframeworks.cc/documentation/gl/ofTexture/#show_unbind) methods which are used for this!
* This includes `ofImage`, `ofVideoGrabber`, and basically anything that contains an `ofTexture`.

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

  ofMesh quadMesh;

  ofParameter<bool> drawWireframe;

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

  // Build the quad mesh.
  quadMesh.setMode(OF_PRIMITIVE_TRIANGLES);

  quadMesh.addVertex(glm::vec3(0, 0, 0));
  quadMesh.addVertex(glm::vec3(640, 0, 0));
  quadMesh.addVertex(glm::vec3(640, 480, 0));
  quadMesh.addVertex(glm::vec3(0, 0, 0));
  quadMesh.addVertex(glm::vec3(640, 480, 0));
  quadMesh.addVertex(glm::vec3(0, 480, 0));

  quadMesh.addTexCoord(glm::vec2(0, 0));
  quadMesh.addTexCoord(glm::vec2(640, 0));
  quadMesh.addTexCoord(glm::vec2(640, 480));
  quadMesh.addTexCoord(glm::vec2(0, 0));
  quadMesh.addTexCoord(glm::vec2(640, 480));
  quadMesh.addTexCoord(glm::vec2(0, 480));

  // Setup the parameters.
  drawWireframe.set("Wireframe?", false);

  // Setup the gui.
  guiPanel.setup("3D World", "settings.json");
  guiPanel.add(drawWireframe);
}

void ofApp::update()
{
  grabber.update();
}

void ofApp::draw()
{
  // Render the mesh.
  grabber.bind();
  if (drawWireframe)
  {
      quadMesh.drawWireframe();
  }
  else
  {
      quadMesh.draw();
  }
  grabber.unbind();

  // Draw the gui.
  guiPanel.draw();
}
```

The last example does essentially what `ofVideoGrabber.draw()` does:

1. Generate a mesh with vertex positions and texture coordinates.
1. Bind the texture provided by the video grabber.
1. Draw the mesh to the screen using the bound texture to set the pixel colors.

<figure style="width:600px;height:400px;display:block;margin:0 auto;">
<iframe src="https://player.vimeo.com/video/141405962?color=ffffff&byline=0&portrait=0" width="640" height="360" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe><script src="https://player.vimeo.com/api/player.js"></script>
<figcaption><i><a href="https://vimeo.com/141405962">Lunar Surface The Incinerator - Installation</a> by <a href="https://www.kimchiandchips.com/works/">Kimchi and Chips</a>.</i></figcaption>
</figure>

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

{{< image src="rs-pointsgray.png" alt="RealSense Points Gray" caption="*RealSense Points Gray*" width="600px" >}}

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
