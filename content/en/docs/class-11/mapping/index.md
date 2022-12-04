---
title: "Mapping"
description: ""
lead: ""
date: 2022-11-28T14:33:17-05:00
lastmod: 2022-11-28T14:33:17-05:00
draft: false
images: []
menu:
  docs:
    parent: "class-11"
    identifier: "mapping"
weight: 999
toc: true
---

OpenCV includes many functions for calibrating and mapping points and images between different spaces.

## Homography

A homography is a transformation that reorients one image to match or fit inside another:

* The two images are usually two points of view of the same subject.
* Common features are found in both images and paired to make a data set.
  {{< image src="https://docs.opencv.org/3.0-beta/_images/homography_findobj.jpg" alt="Feature Matching + Homography to find Objects" caption="[*Feature Matching + Homography to find Objects*](https://docs.opencv.org/3.0-beta/doc/py_tutorials/py_feature2d/py_feature_homography/py_feature_homography.html)" width="600px" >}}
* The data set is then used with the [`cv::findHomography`](https://docs.opencv.org/2.4/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html?highlight=findhomography#findhomography) function, which returns a matrix that allows us to warp one image into the other.
* We can then use this homography in [cv::warpPerspective`](https://docs.opencv.org/2.4/modules/imgproc/doc/geometric_transformations.html#warpperspective), which takes in an image and *warps* it to match the other's perspective.

{{< image src="https://www.learnopencv.com/wp-content/uploads/2016/01/homography-example.jpg" alt="Homography Examples using OpenCV" caption="[*Homography Examples using OpenCV*](https://www.learnopencv.com/homography-examples-using-opencv-python-c/)" width="600px" >}}

This operation can have many uses, like perspective correction, embedding images into one another, or combining many to create large panoramas. The feature pairs can be manually or automatically selected, using OpenCV feature detectors.

<figure style="width:600px;height:420px;display:block;margin:0 auto;">
  <iframe width="600" height="375" src="https://www.youtube.com/embed/GH1p1HtNegY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
  <figcaption><i>video panorama stitching with stabilizing homography estimation</i></figcaption>
</figure>

Let's use homography to map our bouncing ball sketch to a projection surface.

We will start with a couple of changes:

* The canvas will be the size of the projection screen. We will use two macros `PROJECTOR_RESOLUTION_X` and `PROJECTOR_RESOLUTION_Y` to refer to these values across the app.
* We will add a second window for the projector in `main.cpp`.

```cpp
// main.cpp
#include "ofMain.h"
#include "ofApp.h"

int main()
{
  ofGLFWWindowSettings settings;

  settings.setSize(1280, 720);
  settings.setPosition(ofVec2f(100, 100));
  settings.resizable = true;
  shared_ptr<ofAppBaseWindow> mainWindow = ofCreateWindow(settings);

  settings.setSize(PROJECTOR_RESOLUTION_X, PROJECTOR_RESOLUTION_Y);
  settings.setPosition(ofVec2f(ofGetScreenWidth(), 0));
  settings.resizable = false;
  settings.decorated = false;
  settings.shareContextWith = mainWindow;
  shared_ptr<ofAppBaseWindow> secondWindow = ofCreateWindow(settings);
  secondWindow->setVerticalSync(false);

  shared_ptr<ofApp> mainApp(new ofApp);
  ofAddListener(secondWindow->events().draw, mainApp.get(), &ofApp::drawProjector);

  ofRunApp(mainWindow, mainApp);
  ofRunMainLoop();
}
```

* We can update the ball bouncing code to use these macros for the wall coordinates.

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

#include "ofApp.h"

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
  if (pos.x - mass < 0 || pos.x + mass > PROJECTOR_RESOLUTION_X)
  {
    pos.x = ofClamp(pos.x, mass, PROJECTOR_RESOLUTION_X - mass);
    vel.x *= -1;
  }
  if (pos.y - mass < 0 || pos.y + mass > PROJECTOR_RESOLUTION_Y)
  {
    pos.y = ofClamp(pos.y, mass, PROJECTOR_RESOLUTION_Y - mass);
    vel.y *= -1;
  }
}

void ezBall::draw()
{
  ofSetColor(color);
  ofDrawCircle(pos, mass);
}
```

* We will render our bouncing balls canvas into an `ofFbo`, and draw that FBO out to both our main and projector windows.
* We will interact using the mouse on the main screen, so we will have to remap our values in `mouseDragged()`. We only want to add balls when the mouse is dragging on top of the canvas, so we will check if the cursor is in the right spot using [`ofInRange()`](https://openframeworks.cc/documentation/math/ofMath/#!show_ofInRange).

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ezBall.h"

// This must match the display resolution of our projector
#define PROJECTOR_RESOLUTION_X 1920
#define PROJECTOR_RESOLUTION_Y 1080

class ofApp : public ofBaseApp
{
public:
  void setup();
  
  void update();

  void draw();
  void drawProjector(ofEventArgs& args);

  void keyPressed(int key);
  void mouseDragged(int x, int y, int button);
  void mousePressed(int x, int y, int button);

  void addBall(int x, int y);

  std::vector<ezBall> balls;

  ofFbo renderFbo;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofBackground(0);

  renderFbo.allocate(PROJECTOR_RESOLUTION_X, PROJECTOR_RESOLUTION_Y);
}

void ofApp::update()
{
  glm::vec2 gravity = glm::vec2(0, 9.8f);
  renderFbo.begin();
  {
    ofClear(255, 255);

    for (int i = 0; i < balls.size(); i++)
    {
      balls[i].update(gravity);
      balls[i].draw();
    }
  }
  renderFbo.end();
}

void ofApp::draw()
{
  ofSetColor(255);

  // Draw unwarped image on the left.
  renderFbo.draw(0, 0, 640, 360);
}

void ofApp::drawProjector(ofEventArgs& args)
{
  ofBackground(0);
  ofSetColor(255);

  renderFbo.draw(0, 0);
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
  // Only add a ball if we're dragging in the preview window.
  if (ofInRange(x, 0, 640) && ofInRange(y, 0, 360))
  {
    // Remap the ball to the FBO resolution.
    int ballX = ofMap(x, 0, 640, 0, renderFbo.getWidth());
    int ballY = ofMap(y, 0, 360, 0, renderFbo.getHeight());
    addBall(ballX, ballY);
  }
}

void ofApp::mousePressed(int x, int y, int button)
{
  // Simply call the mouseDragged handler.
  mouseDragged(x, y, button);
}
```

Next, let uss add an interface for mapping feature points from one image to the next.

* The points will be normalized (between 0 and 1) to make them independent of image position or resolution. This will allow us to draw them properly in both windows with minimal headaches.
* The source points will never change, we will just use the four corners of our image.
* The destination points can be changed by dragging them with the mouse. We will write a simple system where the point nearest to the mouse gets selected on click.
* The destination points will also be drawn in the projection window, so that we can use them to against actual physical features we are projecting on.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"

#include "ezBall.h"

// This must match the display resolution of your projector
#define PROJECTOR_RESOLUTION_X 1920
#define PROJECTOR_RESOLUTION_Y 1080

class ofApp : public ofBaseApp
{
public:
  void setup();
  
  void update();

  void draw();
  void drawProjector(ofEventArgs& args);

  void keyPressed(int key);
  void mouseDragged(int x, int y, int button);
  void mousePressed(int x, int y, int button);
  void mouseReleased(int x, int y, int button);

  void addBall(int x, int y);

  std::vector<ezBall> balls;

  ofFbo renderFbo;

  std::vector<glm::vec2> srcPoints;
  std::vector<glm::vec2> dstPoints;

  int activePoint;

  ofParameter<bool> adjustMapping;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofBackground(0);

  renderFbo.allocate(PROJECTOR_RESOLUTION_X, PROJECTOR_RESOLUTION_Y);

  srcPoints.push_back(glm::vec2(0, 0));
  srcPoints.push_back(glm::vec2(1, 0));
  srcPoints.push_back(glm::vec2(0, 1));
  srcPoints.push_back(glm::vec2(1, 1));

  dstPoints.push_back(glm::vec2(0, 0));
  dstPoints.push_back(glm::vec2(1, 0));
  dstPoints.push_back(glm::vec2(0, 1));
  dstPoints.push_back(glm::vec2(1, 1));

  activePoint = -1;

  adjustMapping.set("Adjust Mapping", false);

  guiPanel.setup("Homography", "settings.json");
  guiPanel.add(adjustMapping);
}

void ofApp::update()
{
  glm::vec2 gravity = glm::vec2(0, 9.8f);
  renderFbo.begin();
  {
    ofClear(255, 255);

    for (int i = 0; i < balls.size(); i++)
    {
      balls[i].update(gravity);
      balls[i].draw();
    }
  }
  renderFbo.end();
}

void ofApp::draw()
{
  ofSetColor(255);

  // Draw unwarped image on the left.
  renderFbo.draw(0, 0, 640, 360);

  if (adjustMapping)
  {
    // Draw mapping points.
    for (int i = 0; i < srcPoints.size(); i++)
    {
      ofSetColor(0, 0, 255);
      glm::vec2 srcPt = glm::vec2(ofMap(srcPoints[i].x, 0, 1, 0, 640), ofMap(srcPoints[i].y, 0, 1, 0, 360));
      ofDrawCircle(srcPt, 10);

      ofSetColor(255, 0, 0);
      glm::vec2 dstPt = glm::vec2(ofMap(dstPoints[i].x, 0, 1, 640, 1280), ofMap(dstPoints[i].y, 0, 1, 0, 360));
      ofDrawCircle(dstPt, 10);

      ofSetColor(255, 0, 255);
      ofDrawLine(srcPt, dstPt);
    }
  }

  guiPanel.draw();
}

void ofApp::drawProjector(ofEventArgs& args)
{
  ofBackground(0);
  ofSetColor(255);

  renderFbo.draw(0, 0);
  
  if (adjustMapping)
  {
    // Draw mapping dst points.
    for (int i = 0; i < dstPoints.size(); i++)
    {
      ofSetColor(255, 0, 0);
      glm::vec2 dstPt = glm::vec2(dstPoints[i].x * PROJECTOR_RESOLUTION_X, dstPoints[i].y * PROJECTOR_RESOLUTION_Y);
      ofDrawCircle(dstPt, 20);
    }
  }
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
  if (adjustMapping)
  {
    if (activePoint > -1)
    {
      // Move the active Point under the mouse, but stick to edges.
      glm::vec2 normPt = glm::vec2(ofMap(x, 640, 1280, 0, 1, true), ofMap(y, 0, 360, 0, 1, true));
      dstPoints[activePoint] = normPt;
    }
  }
  else
  {
    // Only add a ball if we're dragging in the preview window.
    if (ofInRange(x, 0, 640) && ofInRange(y, 0, 360))
    {
      // Remap the ball to the FBO resolution.
      int ballX = ofMap(x, 0, 640, 0, renderFbo.getWidth());
      int ballY = ofMap(y, 0, 360, 0, renderFbo.getHeight());
      addBall(ballX, ballY);
    }
  }
}

void ofApp::mousePressed(int x, int y, int button)
{
  if (adjustMapping)
  {
    // Try to snap to a dst point.
    for (int i = 0; i < dstPoints.size(); i++)
    {
      glm::vec2 dstPt = glm::vec2(ofMap(dstPoints[i].x, 0, 1, 640, 1280), ofMap(dstPoints[i].y, 0, 1, 0, 360));
      glm::vec2 mousePt = glm::vec2(x, y);
      if (glm::distance(dstPt, mousePt) < 20)
      {
        // Close enough, let's grab this one.
        activePoint = i;
        break;
      }
    }
  }
  else
  {
    // Simply call the mouseDragged handler.
    mouseDragged(x, y, button);
  }
}

void ofApp::mouseReleased(int x, int y, int button)
{
  if (adjustMapping)
  {
    activePoint = -1;
  }
}
```

Finally, we will use these points to first get a homography transformation, then use this transformation to warp our image on the fly.

Note that this might run slowly unless we are running our app in Release mode. This is because our projection FBO is quite large (1920x1080) and we need to read back from the GPU every frame to warp the image on the CPU.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxCv.h"
#include "ofxGui.h"

#include "ezBall.h"

// This must match the display resolution of your projector
#define PROJECTOR_RESOLUTION_X 1920
#define PROJECTOR_RESOLUTION_Y 1080

class ofApp : public ofBaseApp
{
public:
  void setup();
  
  void update();

  void draw();
  void drawProjector(ofEventArgs& args);

  void keyPressed(int key);
  void mouseDragged(int x, int y, int button);
  void mousePressed(int x, int y, int button);
  void mouseReleased(int x, int y, int button);

  void addBall(int x, int y);

  std::vector<ezBall> balls;

  ofFbo renderFbo;
  ofPixels renderPixels;
  ofImage warpedImg;

  std::vector<glm::vec2> srcPoints;
  std::vector<glm::vec2> dstPoints;

  int activePoint;

  cv::Mat homographyMat;
  bool homographyReady;

  ofParameter<bool> adjustMapping;
  ofParameter<bool> projectWarped;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofBackground(0);

  renderFbo.allocate(PROJECTOR_RESOLUTION_X, PROJECTOR_RESOLUTION_Y);
  warpedImg.allocate(PROJECTOR_RESOLUTION_X, PROJECTOR_RESOLUTION_Y, OF_IMAGE_COLOR);

  srcPoints.push_back(glm::vec2(0, 0));
  srcPoints.push_back(glm::vec2(1, 0));
  srcPoints.push_back(glm::vec2(0, 1));
  srcPoints.push_back(glm::vec2(1, 1));

  dstPoints.push_back(glm::vec2(0, 0));
  dstPoints.push_back(glm::vec2(1, 0));
  dstPoints.push_back(glm::vec2(0, 1));
  dstPoints.push_back(glm::vec2(1, 1));

  activePoint = -1;
  homographyReady = false;

  adjustMapping.set("Adjust Mapping", false);
  projectWarped.set("Project Warped", true);

  guiPanel.setup("Homography", "settings.json");
  guiPanel.add(adjustMapping);
  guiPanel.add(projectWarped);
}

void ofApp::update()
{
  if (adjustMapping)
  {
    // Copy points from glm to cv format.
    std::vector<cv::Point2f> cvSrcPoints;
    std::vector<cv::Point2f> cvDstPoints;
    for (int i = 0; i < srcPoints.size(); i++) 
    {
      // Scale points to projector dimensions.
      cvSrcPoints.push_back(cv::Point2f(srcPoints[i].x * PROJECTOR_RESOLUTION_X, srcPoints[i].y * PROJECTOR_RESOLUTION_Y));
      cvDstPoints.push_back(cv::Point2f(dstPoints[i].x * PROJECTOR_RESOLUTION_X, dstPoints[i].y * PROJECTOR_RESOLUTION_Y));
    }

    // Generate a homography from the two sets of points.
    homographyMat = cv::findHomography(cv::Mat(cvSrcPoints), cv::Mat(cvDstPoints));
    homographyReady = true;
  }

  glm::vec2 gravity = glm::vec2(0, 9.8f);
  renderFbo.begin();
  {
    ofClear(255, 255);

    for (int i = 0; i < balls.size(); i++)
    {
      balls[i].update(gravity);
      balls[i].draw();
    }
  }
  renderFbo.end();

  if (homographyReady) 
  {
    // Read the FBO to pixels.
    renderFbo.readToPixels(renderPixels);

    // Warp the pixels into a new image.
    warpedImg.setFromPixels(renderPixels);
    ofxCv::warpPerspective(renderPixels, warpedImg, homographyMat, CV_INTER_LINEAR);
    warpedImg.update();
  }
}

void ofApp::draw()
{
  ofSetColor(255);

  // Draw unwarped image on the left.
  renderFbo.draw(0, 0, 640, 360);

  if (homographyReady)
  {
    // Draw warped image on the right.
    warpedImg.draw(640, 0, 640, 360);
  }

  if (adjustMapping)
  {
    // Draw mapping points.
    for (int i = 0; i < srcPoints.size(); i++)
    {
      ofSetColor(0, 0, 255);
      glm::vec2 srcPt = glm::vec2(ofMap(srcPoints[i].x, 0, 1, 0, 640), ofMap(srcPoints[i].y, 0, 1, 0, 360));
      ofDrawCircle(srcPt, 10);

      ofSetColor(255, 0, 0);
      glm::vec2 dstPt = glm::vec2(ofMap(dstPoints[i].x, 0, 1, 640, 1280), ofMap(dstPoints[i].y, 0, 1, 0, 360));
      ofDrawCircle(dstPt, 10);

      ofSetColor(255, 0, 255);
      ofDrawLine(srcPt, dstPt);
    }
  }

  guiPanel.draw();
}

void ofApp::drawProjector(ofEventArgs& args)
{
  ofBackground(0);
  ofSetColor(255);

  if (homographyReady && projectWarped)
  {
    warpedImg.draw(0, 0);
  }
  else
  {
    renderFbo.draw(0, 0);
  }

  if (adjustMapping)
  {
    // Draw mapping dst points.
    for (int i = 0; i < dstPoints.size(); i++)
    {
      ofSetColor(255, 0, 0);
      glm::vec2 dstPt = glm::vec2(dstPoints[i].x * PROJECTOR_RESOLUTION_X, dstPoints[i].y * PROJECTOR_RESOLUTION_Y);
      ofDrawCircle(dstPt, 20);
    }
  }
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
  if (adjustMapping)
  {
    if (activePoint > -1)
    {
      // Move the active Point under the mouse, but stick to edges.
      glm::vec2 normPt = glm::vec2(ofMap(x, 640, 1280, 0, 1, true), ofMap(y, 0, 360, 0, 1, true));
      dstPoints[activePoint] = normPt;
    }
  }
  else
  {
    // Only add a ball if we're dragging in the preview window.
    if (ofInRange(x, 0, 640) && ofInRange(y, 0, 360))
    {
      // Remap the ball to the FBO resolution.
      int ballX = ofMap(x, 0, 640, 0, renderFbo.getWidth());
      int ballY = ofMap(y, 0, 360, 0, renderFbo.getHeight());
      addBall(ballX, ballY);
    }
  }
}

void ofApp::mousePressed(int x, int y, int button)
{
  if (adjustMapping)
  {
    // Try to snap to a dst point.
    for (int i = 0; i < dstPoints.size(); i++)
    {
      glm::vec2 dstPt = glm::vec2(ofMap(dstPoints[i].x, 0, 1, 640, 1280), ofMap(dstPoints[i].y, 0, 1, 0, 360));
      glm::vec2 mousePt = glm::vec2(x, y);
      if (glm::distance(dstPt, mousePt) < 20)
      {
        // Close enough, let's grab this one.
        activePoint = i;
        break;
      }
    }
  }
  else
  {
    mouseDragged(x, y, button);
  }
}

void ofApp::mouseReleased(int x, int y, int button)
{
  if (adjustMapping)
  {
    activePoint = -1;
  }
}
```

{{< alert context="info" icon="✌️" >}}
**Homography vs Quad Warping**

You may be wondering why we are not just binding our FBO texture to a quad mesh, positioning the corner points and rendering that out instead of going through all the homography steps outlined above.

The main difference is that the homography operation is going to look more accurate because it is performing a perspective shift; it is looking at the image from a different *virtual* point of view, which is essentially what we are doing in the *physical* world.

Warping a quad will just perform a simple interpolation on the image pixels to make them fit into the new shape. It might look fine for subtle transformations, but will break down quickly as the transformation becomes more extreme.

<figure style="width:600px;height:200px;display:block;margin:0 auto;">
<video src="homography-warp.mp4" controls width="100%"></video>
<figcaption><i>Homography (left) / Quad Warp (right)</i></figcaption>
</figure>
{{< /alert >}}

## World to Projector Mapping

Homography is useful for 2D mapping (i.e. images to images), but we can also attempt to map the 3D space we are projecting onto back into the projector image.

<figure style="width:600px;height:420px;display:block;margin:0 auto;">
  <iframe width="600" height="375" src="https://www.youtube.com/embed/CE1B7tdGCw0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
  <figcaption><i>UCLA's Augmented Reality Sandbox</i></figcaption>
</figure>

The idea is similar:

* We will collect pairs of corresponding points from both spaces. However, since one space is our world, we will need its points in 3D. We will use a depth sensor for this.
* We can then pass this data set to a solver function, which will give us a transformation we can apply to other points in the same space, and have them *project* to an appropriate position on screen.

### Model View Projection

We have already been doing something similar when rendering 3D objects on screen, like point clouds. You may have heard of the Model View Projection Matrix (or MVP) when working with computer graphics.

* The Model View Projection is actually a stack of three matrices that are used to transform a 3D point from its local space to screen space.
* The *model matrix* maps a point from its local space to the world space.
* The *view matrix* maps a point from world space to camera space (from the point of view of the camera).
* The *projection matrix* maps a point from camera space to clip space, which is essentially what the camera projects onto a surface. Depending on the camera parameters, this projection can have perspective or be orthographic.

{{< image src="https://www.maa.org/sites/default/files/images/cms_upload/figure256786.jpg" alt="Geometric Photo Manipulation - Projections" caption="[*Geometric Photo Manipulation - Projections*](https://www.maa.org/press/periodicals/loci/joma/geometric-photo-manipulation-projections)" width="600px" >}}

What we are essentially doing is coming up with a similar matrix, but with an external camera and projector.

[Model View Projection](https://jsantell.com/model-view-projection) is a good reference for in-depth info.

### Pinhole Camera Model

We use a pinhole camera model to calculate this projection. Using a perspective model, the line of sight from the camera to a 3D point will intersect a plane, which we can consider our canvas. The position at which it intersects the plane is its projection in 2D space.

{{< image src="https://docs.opencv.org/2.4/_images/pinhole_camera_model.png" alt="Camera Calibration and 3D Reconstruction" caption="[*Camera Calibration and 3D Reconstruction*](https://docs.opencv.org/2.4/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html)" width="600px" >}}

### ofxKinectProjectorToolkit

[ofxKinectProjectorToolkit](https://github.com/genekogan/ofxKinectProjectorToolkit) is one of the many addons available for OF to create a correspondence between the 3D world and a 2D projector.

* This addon was originally written by Gene Kogan but it's a little out of date. I've made a fork [here](https://github.com/prisonerjohn/ofxKinectProjectorToolkit/tree/feature/refactor-0.10) with updates which should get you started faster.
* As the name implies, this addon is meant to work with the Kinect sensor. However, the calibration functions are sensor-agnostic. You just need to provide point pairs and it does the rest.
* A chessboard pattern is used for feature detection. The chessboard is commonly used in OpenCV because it is high contrast and easy to track. The [`cv::findChessboardCorners()`](https://docs.opencv.org/2.4/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html#findchessboardcorners) function is used to track the intersection points in the pattern.
* The chessboard is drawn out of the projector. Because we are drawing the points, we know their 2D position on screen, in projector space.
* The chessboard is detected using the depth sensor's color camera, which is looking at the projection surface. The tracked points are then fed to the sensor's world coordinate mapper to extract corresponding 3D world points.
* The two sets of points are then fed to a solver, which determines the transformation matrix from one space to the other.
* `ofxKinectProjectorToolkit` uses [dlib](http://dlib.net/) for calibration, which is another commonly used image processing library.
* If we were to use OpenCV, we would probably use the [`cv::calibrateCamera()`](https://docs.opencv.org/2.4/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html#calibratecamera) function.

### Projected Blob Clipping

The following is UNTESTED code that uses this addon to attempt to segment out a blob from a projected image. We'll try it in class and see what happens :)

Here is the version for Kinect.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxCv.h"
#include "ofxGui.h"
#include "ofxKinect.h"
#include "ofxKinectProjectorToolkit.h"

#define CHESSBOARD_COLS 5
#define CHESSBOARD_ROWS 4

// This must match the display resolution of your projector
#define PROJECTOR_RESOLUTION_X 1920
#define PROJECTOR_RESOLUTION_Y 1080

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  void keyPressed(int key);
  //void keyReleased(int key);
  //void mouseMoved(int x, int y);
  void mouseDragged(int x, int y, int button);
  void mousePressed(int x, int y, int button);
  //void mouseReleased(int x, int y, int button);
  //void mouseEntered(int x, int y);
  //void mouseExited(int x, int y);
  //void windowResized(int w, int h);
  //void dragEvent(ofDragInfo dragInfo);
  //void gotMessage(ofMessage msg);

  void drawProjector(ofEventArgs& args);

  void renderChessboard();
  void renderTestPoint(glm::vec2 projectedPoint);
  void renderContours();

  void addPointPairs();

  void calibrateSpaces();
  void saveCalibration();
  void loadCalibration();

  ofxKinect kinect;
  ofxKinectProjectorToolkit kpToolkit;

  int deviceWidth;
  int deviceHeight;

  ofFbo fboProjection;

  ofImage colorImg;
  cv::Mat colorMat;

  vector<glm::vec2> currProjectorPoints;
  vector<cv::Point2f> cvPoints;
  vector<glm::vec3> pairsWorld;
  vector<glm::vec2> pairsProjector;

  glm::vec2 testPoint;

  ofImage thresholdImg;
  ofxCv::ContourFinder contourFinder;

  ofParameter<int> appMode;

  ofParameter<float> chessboardX;
  ofParameter<float> chessboardY;
  ofParameter<int> chessboardSize;
  ofParameter<void> addPairs;
  ofParameter<void> calibrate;
  ofParameter<void> saveCalib;
  ofParameter<void> loadCalib;

  ofParameter<float> nearThreshold;
  ofParameter<float> farThreshold;
  ofParameter<float> minArea;
  ofParameter<float> maxArea;
  ofParameter<bool> flipX;
  ofParameter<bool> flipY;

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

  appMode.set("App Mode", 0, 0, 2);

  chessboardX.set("Chessboard X", 0.5, 0.0, 1.0);
  chessboardY.set("Chessboard Y", 0.5, 0.0, 1.0);
  chessboardSize.set("Chessboard Size", 300, 20, 1280);

  addPairs.set("Add Pairs");
  addPairs.addListener(this, &ofApp::addPointPairs);
  calibrate.set("Calibrate");
  calibrate.addListener(this, &ofApp::calibrateSpaces);
  saveCalib.set("Save Calib");
  saveCalib.addListener(this, &ofApp::saveCalibration);
  loadCalib.set("Load Calib");
  loadCalib.addListener(this, &ofApp::loadCalibration);

  nearThreshold.set("Near Threshold", 0.01f, 0.0f, 0.1f);
  farThreshold.set("Far Threshold", 0.02f, 0.0f, 0.1f);
  minArea.set("Min Area", 0.01f, 0, 0.5f);
  maxArea.set("Max Area", 0.05f, 0, 0.5f);

  flipX.set("Flip X", false);
  flipY.set("Flip Y", false);

  fboProjection.allocate(PROJECTOR_RESOLUTION_X, PROJECTOR_RESOLUTION_Y, GL_RGBA);

  guiPanel.setup("RS Projection", "settings.json");
  guiPanel.add(appMode);
  guiPanel.add(chessboardX);
  guiPanel.add(chessboardY);
  guiPanel.add(chessboardSize);
  guiPanel.add(addPairs);
  guiPanel.add(calibrate);
  guiPanel.add(saveCalib);
  guiPanel.add(loadCalib);
  guiPanel.add(nearThreshold);
  guiPanel.add(farThreshold);
  guiPanel.add(minArea);
  guiPanel.add(maxArea);
  guiPanel.add(flipX);
  guiPanel.add(flipY);

  // Start the Kinect context.
  kinect.setRegistration(true);
  kinect.init();
  kinect.open();

  deviceWidth = kinect.getWidth();
  deviceHeight = kinect.getHeight();
  colorImg.allocate(deviceWidth, deviceHeight, OF_IMAGE_COLOR);
}


void ofApp::update()
{
  kinect.update();

  if (kinect.isFrameNew())
  {
    colorImg.setFromPixels(kinect.getPixels());
    if (appMode == 0) // Searching.
    {
      renderChessboard();

      colorMat = ofxCv::toCv(colorImg.getPixels());
      cv::Size patternSize = cv::Size(CHESSBOARD_COLS - 1, CHESSBOARD_ROWS - 1);
      int chessFlags = cv::CALIB_CB_ADAPTIVE_THRESH + cv::CALIB_CB_FAST_CHECK;
      bool foundChessboard = cv::findChessboardCorners(colorMat, patternSize, cvPoints, chessFlags);
      if (foundChessboard)
      {
        cv::Mat grayMat;
        cv::cvtColor(colorMat, grayMat, CV_RGB2GRAY);
        cv::cornerSubPix(grayMat, cvPoints, cv::Size(11, 11), cv::Size(-1, -1), cv::TermCriteria(CV_TERMCRIT_EPS + CV_TERMCRIT_ITER, 30, 0.1));
        cv::drawChessboardCorners(colorMat, patternSize, cv::Mat(cvPoints), foundChessboard);
        colorImg.update();
      }
    }
    else if (appMode == 1) // Testing.
    {
      // Map points in both world space and projector space and draw them.
      // If they are calibrated correctly they should be drawn on top of one another.
      glm::vec2 clampedTestPoint = glm::vec2(
          ofClamp(testPoint.x, 0, deviceWidth - 1),
          ofClamp(testPoint.y, 0, deviceHeight - 1));
      glm::vec3 worldPoint = kinect.getWorldCoordinateAt(clampedTestPoint.x, clampedTestPoint.y);
      glm::vec2 projectedPoint = kpToolkit.getProjectedPoint(worldPoint);

      renderTestPoint(projectedPoint);
    }
    else // Rendering.
    {
      // Threshold the depth.
      ofFloatPixels rawDepthPix = kinect.getRawDepthPixels();
      ofFloatPixels thresholdNear, thresholdFar, thresholdResult;
      ofxCv::threshold(rawDepthPix, thresholdNear, nearThreshold);
      ofxCv::threshold(rawDepthPix, thresholdFar, farThreshold, true);
      ofxCv::bitwise_and(thresholdNear, thresholdFar, thresholdResult);
      thresholdImg.setFromPixels(thresholdResult);

      // Find contours.
      contourFinder.setMinAreaNorm(minArea);
      contourFinder.setMaxAreaNorm(maxArea);
      contourFinder.findContours(thresholdImg);

      renderContours();
    }
  }
}

void ofApp::draw()
{
  ofSetColor(255);
  colorImg.draw(0, 0);

  kinect.getDepthTexture().draw(colorImg.getWidth(), 0);

  if (thresholdImg.isAllocated())
  {
    ofPushMatrix();
    ofTranslate(colorImg.getWidth(), kinect.getDepthTexture().getHeight());
    {
      thresholdImg.draw(0, 0);
      contourFinder.draw();
    }
    ofPopMatrix();
  }

  if (appMode == 0) // Searching.
  {
    // Use a string stream to print a multi-line message.
    std::ostringstream oss;
    oss << "SEARCHING MODE" << std::endl
        << "Position the chessboard by dragging the mouse over the RGB image." << std::endl
        << "Adjust the size of the chessboard using LEFT / RIGHT or the GUI sliders." << std::endl
        << "When the pattern is recognized, hit SPACE to save a set of point-pairs." << std::endl
        << "Once a few point-pairs have been detected, calibrate using the GUI button." << std::endl
        << std::endl
        << ofToString(pairsWorld.size()) << " point-pairs collected.";

    ofSetColor(255);
    ofDrawBitmapStringHighlight(oss.str(), 10, 380);
  }
  else if (appMode == 1) // Testing.
  {
    // Use a string stream to print a multi-line message.
    std::ostringstream oss;
    oss << "TESTING MODE" << std::endl
        << "Click on the RGB image to test a calibrated point." << std::endl
        << "The world point will be drawn in RED, the projected point will be drawn in GREEN." << std::endl
        << "If the calibration is successful, both points will be drawn on top of each other." << std::endl
        << "Save the calibration using the GUI button.";

    ofSetColor(255);
    ofDrawBitmapStringHighlight(oss.str(), 10, 380);

    // Draw the test point on screen.
    ofSetColor(255, 0, 0);
    float pointSize = ofMap(cos(ofGetFrameNum() * 0.1), -1, 1, 3, 40);
    ofDrawCircle(testPoint.x, testPoint.y, pointSize);
  }
  else // Rendering.
  {
    // Use a string stream to print a multi-line message.
    std::ostringstream oss;
    oss << "RENDERING MODE" << std::endl
        << "Adjust the depth threshold using the GUI sliders." << std::endl
        << "The thresholded silhouette will mask the rest of the projected image.";
  }

  guiPanel.draw();
}

void ofApp::drawProjector(ofEventArgs& args)
{
  ofBackground(255);
  ofSetColor(255);
  fboProjection.draw(0, 0);
}

void ofApp::keyPressed(int key)
{
  if (key == ' ')
  {
    addPointPairs();
  }
  else if (key == 'c')
  {
    calibrateSpaces();
  }
  else if (key == 's')
  {
    saveCalibration();
  }
  else if (key == 'l')
  {
    loadCalibration();
  }
}

void ofApp::mousePressed(int x, int y, int button)
{
  if (appMode == 0) // Searching
  {
    if (ofGetMousePressed() &&
        ofInRange(x, 0, deviceWidth) &&
        ofInRange(y, 0, deviceHeight))
    {
      // Save the normalized point position.
      chessboardX = ofMap(x, 0, deviceWidth, 0, 1);
      chessboardY = ofMap(y, 0, deviceHeight, 0, 1);
    }
  }
  else if (appMode == 1) // Testing
  {
    testPoint = glm::vec2(x, y);
  }
  else
  {
    // ?
  }
}

void ofApp::mouseDragged(int x, int y, int button)
{
  if (appMode == 0) // Searching
  {
    if (ofGetMousePressed() &&
        ofInRange(x, 0, deviceWidth) &&
        ofInRange(y, 0, deviceHeight))
    {
      // Save the normalized point position.
      chessboardX = ofMap(x, 0, deviceWidth, 0, 1);
      chessboardY = ofMap(y, 0, deviceHeight, 0, 1);
    }
  }
  else if (appMode == 1) // Testing
  {
    testPoint = glm::vec2(x, y);
  }
}

void ofApp::renderChessboard()
{
  float cellSize = chessboardSize / CHESSBOARD_COLS;

  currProjectorPoints.clear();

  fboProjection.begin();
  {
    // Remap top-left to projection space.
    int boardX = ofMap(chessboardX, 0, 1, 0, fboProjection.getWidth());
    int boardY = ofMap(chessboardY, 0, 1, 0, fboProjection.getHeight());

    // Clear white and draw black cells.
    ofClear(255, 0);
    ofSetColor(0);

    for (int j = 0; j < CHESSBOARD_ROWS; j++)
    {
      for (int i = 0; i < CHESSBOARD_COLS; i++)
      {
        int cellX = boardX + i * cellSize;
        int cellY = boardY + j * cellSize;

        if ((i + j) % 2 == 0)
        {
          // Only draw black cells.
          ofDrawRectangle(cellX, cellY, cellSize, cellSize);
        }

        if (i > 0 && j > 0)
        {
          // Add normalized intersection points to the list.
          float normX = ofMap(cellX, 0, fboProjection.getWidth(), 0, 1);
          float normY = ofMap(cellY, 0, fboProjection.getHeight(), 0, 1);
          currProjectorPoints.push_back(glm::vec2(normX, normY));
        }
      }
    }
  }
  fboProjection.end();
}

void ofApp::renderTestPoint(glm::vec2 projectedPoint)
{
  float pointSize = ofMap(sin(ofGetFrameNum() * 0.1), -1, 1, 3, 40);

  fboProjection.begin();
  {
    ofBackground(255);

    // Point is normalized, so it needs to be mapped to the projector size.
    float projX = ofMap(projectedPoint.x, 0, 1, 0, fboProjection.getWidth());
    float projY = ofMap(projectedPoint.y, 0, 1, 0, fboProjection.getHeight());

    ofSetColor(0, 255, 0);
    ofDrawCircle(projX, projY, pointSize);
  }
  fboProjection.end();
}

void ofApp::renderContours()
{
  fboProjection.begin();
  {
    // Clear white and draw black contours.
    ofClear(255, 0);
    ofSetColor(0);

    for (int i = 0; i < contourFinder.size(); i++)
    {
      // Map contour using calibration and draw to main window
      ofBeginShape();
      std::vector<cv::Point> points = contourFinder.getContour(i);
      for (int j = 0; j < points.size(); j++)
      {
        glm::vec3 worldPoint = kinect.getWorldCoordinateAt(points[j].x, points[j].y);
        if (worldPoint.z > 0)
        {
          glm::vec2 projectedPoint = kpToolkit.getProjectedPoint(worldPoint);
          if (ofInRange(projectedPoint.x, 0, 1) && ofInRange(projectedPoint.y, 0, 1))
          {
            if (flipX)
            {
              projectedPoint.x = 1.0 - projectedPoint.x;
            }
            if (flipY)
            {
              projectedPoint.y = 1.0 - projectedPoint.y;
            }
            ofVertex(projectedPoint.x * fboProjection.getWidth(), projectedPoint.y * fboProjection.getHeight());
            //ofLog() << "Adding world " << worldPoint << " // point " << projectedPoint << " // proj " << (projectedPoint.x * fboProjection.getWidth()) << ", " << (projectedPoint.y * fboProjection.getHeight());
          }
        }
      }
      ofEndShape();
    }
  }
  fboProjection.end();
}

void ofApp::addPointPairs()
{
  // Count the number of world points.
  int numDepthPoints = 0;
  for (int i = 0; i < cvPoints.size(); i++)
  {
    glm::vec3 worldPoint = kinect.getWorldCoordinateAt(cvPoints[i].x, cvPoints[i].y);
    if (worldPoint.z > 0)
    {
      numDepthPoints++;
    }
  }

  int chessboardTotal = (CHESSBOARD_COLS - 1) * (CHESSBOARD_ROWS - 1);
  if (numDepthPoints != chessboardTotal)
  {
    ofLogError(__FUNCTION__) << "Only found " << numDepthPoints << " / " << chessboardTotal << " points!";
    return;
  }

  // Found all chessboard points in the world, add both sets to the lists we will use for calibration.
  for (int i = 0; i < cvPoints.size(); i++)
  {
    glm::vec3 worldPoint = kinect.getWorldCoordinateAt(cvPoints[i].x, cvPoints[i].y);
    pairsWorld.push_back(worldPoint);
    pairsProjector.push_back(currProjectorPoints[i]);
    ofLogNotice(__FUNCTION__) << "Adding pair i: " << worldPoint << " => " << currProjectorPoints[i];
  }

  ofLogNotice(__FUNCTION__) << "Added " << chessboardTotal << " point-pairs.";
}

void ofApp::calibrateSpaces()
{
  kpToolkit.calibrate(pairsWorld, pairsProjector);
  appMode = 1;

  pairsWorld.clear();
  pairsProjector.clear();
}

void ofApp::saveCalibration()
{
  if (kpToolkit.saveCalibration("calibration.json"))
  {
    ofLogNotice(__FUNCTION__) << "Calibration saved!";
  }
}

void ofApp::loadCalibration()
{
  if (kpToolkit.loadCalibration("calibration.json"))
  {
    ofLogNotice(__FUNCTION__) << "Calibration loaded!";
    appMode = 2;
  }
}
```

And the version for RealSense.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxCv.h"
#include "ofxGui.h"
#include "ofxKinectProjectorToolkit.h"
#include "ofxRealSense2.h"

#define CHESSBOARD_COLS 5
#define CHESSBOARD_ROWS 4

// This must match the display resolution of your projector
#define PROJECTOR_RESOLUTION_X 1920
#define PROJECTOR_RESOLUTION_Y 1080

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  void keyPressed(int key);
  //void keyReleased(int key);
  //void mouseMoved(int x, int y);
  void mouseDragged(int x, int y, int button);
  void mousePressed(int x, int y, int button);
  //void mouseReleased(int x, int y, int button);
  //void mouseEntered(int x, int y);
  //void mouseExited(int x, int y);
  //void windowResized(int w, int h);
  //void dragEvent(ofDragInfo dragInfo);
  //void gotMessage(ofMessage msg);

  void deviceAdded(std::string& serialNumber);

  void drawProjector(ofEventArgs& args);

  void renderChessboard();
  void renderTestPoint(glm::vec2 projectedPoint);
  void renderContours();

  void addPointPairs();

  void calibrateSpaces();
  void saveCalibration();
  void loadCalibration();

  ofxRealSense2::Context rsContext;
  ofxKinectProjectorToolkit kpToolkit;

  int deviceWidth;
  int deviceHeight;

  ofFbo fboProjection;

  ofImage colorImg;
  cv::Mat colorMat;

  vector<glm::vec2> currProjectorPoints;
  vector<cv::Point2f> cvPoints;
  vector<glm::vec3> pairsWorld;
  vector<glm::vec2> pairsProjector;

  glm::vec2 testPoint;

  ofImage thresholdImg;
  ofxCv::ContourFinder contourFinder;

  ofParameter<int> appMode;

  ofParameter<float> chessboardX;
  ofParameter<float> chessboardY;
  ofParameter<int> chessboardSize;
  ofParameter<void> addPairs;
  ofParameter<void> calibrate;
  ofParameter<void> saveCalib;
  ofParameter<void> loadCalib;

  ofParameter<float> nearThreshold;
  ofParameter<float> farThreshold;
  ofParameter<float> minArea;
  ofParameter<float> maxArea;
  ofParameter<bool> flipX;
  ofParameter<bool> flipY;

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

    appMode.set("App Mode", 0, 0, 2);

    chessboardX.set("Chessboard X", 0.5, 0.0, 1.0);
    chessboardY.set("Chessboard Y", 0.5, 0.0, 1.0);
    chessboardSize.set("Chessboard Size", 300, 20, 1280);

    addPairs.set("Add Pairs");
    addPairs.addListener(this, &ofApp::addPointPairs);
    calibrate.set("Calibrate");
    calibrate.addListener(this, &ofApp::calibrateSpaces);
    saveCalib.set("Save Calib");
    saveCalib.addListener(this, &ofApp::saveCalibration);
    loadCalib.set("Load Calib");
    loadCalib.addListener(this, &ofApp::loadCalibration);

    nearThreshold.set("Near Threshold", 0.01f, 0.0f, 0.1f);
    farThreshold.set("Far Threshold", 0.02f, 0.0f, 0.1f);
    minArea.set("Min Area", 0.01f, 0, 0.5f);
    maxArea.set("Max Area", 0.05f, 0, 0.5f);

    flipX.set("Flip X", false);
    flipY.set("Flip Y", false);

    fboProjection.allocate(PROJECTOR_RESOLUTION_X, PROJECTOR_RESOLUTION_Y, GL_RGBA);

    guiPanel.setup("RS Projection", "settings.json");
    guiPanel.add(appMode);
    guiPanel.add(chessboardX);
    guiPanel.add(chessboardY);
    guiPanel.add(chessboardSize);
    guiPanel.add(addPairs);
    guiPanel.add(calibrate);
    guiPanel.add(saveCalib);
    guiPanel.add(loadCalib);
    guiPanel.add(nearThreshold);
    guiPanel.add(farThreshold);
    guiPanel.add(minArea);
    guiPanel.add(maxArea);
    guiPanel.add(flipX);
    guiPanel.add(flipY);

    // Start the RealSense context.
    // Devices are added in the deviceAdded() callback function.
    ofAddListener(rsContext.deviceAddedEvent, this, &ofApp::deviceAdded);
    rsContext.setup(false);
}

void ofApp::deviceAdded(std::string& serialNumber)
{
    ofLogNotice(__FUNCTION__) << "Starting device " << serialNumber;
    auto device = rsContext.getDevice(serialNumber);
    device->enableInfrared();
    device->enableDepth();
    device->enableColor();
    device->startPipeline();

    // Work in device depth space (should be 640x360).
    device->alignMode = ofxRealSense2::Device::Align::Color;
    deviceWidth = device->getColorPix().getWidth();
    deviceHeight = device->getColorPix().getHeight();
    colorImg.allocate(deviceWidth, deviceHeight, OF_IMAGE_COLOR);

    // Uncomment this to add the device specific settings to the GUI.
    //guiPanel.add(device->params);
}

//--------------------------------------------------------------
void ofApp::update()
{
    rsContext.update();

    std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
    if (rsDevice)
    {
        colorImg.setFromPixels(rsDevice->getColorPix());
        if (appMode == 0) // Searching.
        {
            renderChessboard();

            colorMat = ofxCv::toCv(colorImg.getPixels());
            cv::Size patternSize = cv::Size(CHESSBOARD_COLS - 1, CHESSBOARD_ROWS - 1);
            int chessFlags = cv::CALIB_CB_ADAPTIVE_THRESH + cv::CALIB_CB_FAST_CHECK;
            bool foundChessboard = cv::findChessboardCorners(colorMat, patternSize, cvPoints, chessFlags);
            if (foundChessboard)
            {
                cv::Mat grayMat;
                cv::cvtColor(colorMat, grayMat, CV_RGB2GRAY);
                cv::cornerSubPix(grayMat, cvPoints, cv::Size(11, 11), cv::Size(-1, -1), cv::TermCriteria(CV_TERMCRIT_EPS + CV_TERMCRIT_ITER, 30, 0.1));
                cv::drawChessboardCorners(colorMat, patternSize, cv::Mat(cvPoints), foundChessboard);
                colorImg.update();
            }
        }
        else if (appMode == 1) // Testing.
        {
            // Map points in both world space and projector space and draw them.
            // If they are calibrated correctly they should be drawn on top of one another.
            glm::vec2 clampedTestPoint = glm::vec2(
                ofClamp(testPoint.x, 0, deviceWidth - 1),
                ofClamp(testPoint.y, 0, deviceHeight - 1));
            glm::vec3 worldPoint = rsDevice->getWorldPosition(clampedTestPoint.x, clampedTestPoint.y);
            glm::vec2 projectedPoint = kpToolkit.getProjectedPoint(worldPoint);

            renderTestPoint(projectedPoint);
        }
        else // Rendering.
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

            renderContours();
        }
    }
}

void ofApp::draw()
{
    std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
    if (rsDevice)
    {
        ofSetColor(255);
        colorImg.draw(0, 0);

        rsDevice->getDepthTex().draw(colorImg.getWidth(), 0);

        if (thresholdImg.isAllocated())
        {
            ofPushMatrix();
            ofTranslate(colorImg.getWidth(), rsDevice->getDepthTex().getHeight());
            {
                thresholdImg.draw(0, 0);
                contourFinder.draw();
            }
            ofPopMatrix();
        }

        if (appMode == 0) // Searching.
        {
            // Use a string stream to print a multi-line message.
            std::ostringstream oss;
            oss << "SEARCHING MODE" << std::endl
                << "Position the chessboard by dragging the mouse over the RGB image." << std::endl
                << "Adjust the size of the chessboard using LEFT / RIGHT or the GUI sliders." << std::endl
                << "When the pattern is recognized, hit SPACE to save a set of point-pairs." << std::endl
                << "Once a few point-pairs have been detected, calibrate using the GUI button." << std::endl
                << std::endl
                << ofToString(pairsWorld.size()) << " point-pairs collected.";

            ofSetColor(255);
            ofDrawBitmapStringHighlight(oss.str(), 10, 380);
        }
        else if (appMode == 1) // Testing.
        {
            // Use a string stream to print a multi-line message.
            std::ostringstream oss;
            oss << "TESTING MODE" << std::endl
                << "Click on the RGB image to test a calibrated point." << std::endl
                << "The world point will be drawn in RED, the projected point will be drawn in GREEN." << std::endl
                << "If the calibration is successful, both points will be drawn on top of each other." << std::endl
                << "Save the calibration using the GUI button.";

            ofSetColor(255);
            ofDrawBitmapStringHighlight(oss.str(), 10, 380);

            // Draw the test point on screen.
            ofSetColor(255, 0, 0);
            float pointSize = ofMap(cos(ofGetFrameNum() * 0.1), -1, 1, 3, 40);
            ofDrawCircle(testPoint.x, testPoint.y, pointSize);
        }
        else // Rendering.
        {
            // Use a string stream to print a multi-line message.
            std::ostringstream oss;
            oss << "RENDERING MODE" << std::endl
                << "Adjust the depth threshold using the GUI sliders." << std::endl
                << "The thresholded silhouette will mask the rest of the projected image.";
        }
    }

    guiPanel.draw();
}

void ofApp::drawProjector(ofEventArgs& args)
{
    ofBackground(255);
    ofSetColor(255);
    fboProjection.draw(0, 0);
}

void ofApp::keyPressed(int key)
{
    if (key == ' ')
    {
        addPointPairs();
    }
    else if (key == 'c') 
    {
        calibrateSpaces();
    }
    else if (key == 's')
    {
        saveCalibration();
    }
    else if (key == 'l')
    {
        loadCalibration();
    }
}

void ofApp::mousePressed(int x, int y, int button)
{
    if (appMode == 0) // Searching
    {
        if (ofGetMousePressed() &&
            ofInRange(x, 0, deviceWidth) &&
            ofInRange(y, 0, deviceHeight))
        {
            // Save the normalized point position.
            chessboardX = ofMap(x, 0, deviceWidth, 0, 1);
            chessboardY = ofMap(y, 0, deviceHeight, 0, 1);
        }
    }
    else if (appMode == 1) // Testing
    {
        testPoint = glm::vec2(x, y);
    }
    else
    {

    }
}

void ofApp::mouseDragged(int x, int y, int button)
{
    if (appMode == 0) // Searching
    {
        if (ofGetMousePressed() &&
            ofInRange(x, 0, deviceWidth) &&
            ofInRange(y, 0, deviceHeight))
        {
            // Save the normalized point position.
            chessboardX = ofMap(x, 0, deviceWidth, 0, 1);
            chessboardY = ofMap(y, 0, deviceHeight, 0, 1);
        }
    }
    else if (appMode == 1) // Testing
    {
        testPoint = glm::vec2(x, y);
    }
}

void ofApp::renderChessboard()
{
    float cellSize = chessboardSize / CHESSBOARD_COLS;

    currProjectorPoints.clear();

    fboProjection.begin();
    {
        // Remap top-left to projection space.
        int boardX = ofMap(chessboardX, 0, 1, 0, fboProjection.getWidth());
        int boardY = ofMap(chessboardY, 0, 1, 0, fboProjection.getHeight());

        // Clear white and draw black cells.
        ofClear(255, 0);
        ofSetColor(0);

        for (int j = 0; j < CHESSBOARD_ROWS; j++)
        {
            for (int i = 0; i < CHESSBOARD_COLS; i++)
            {
                int cellX = boardX + i * cellSize;
                int cellY = boardY + j * cellSize;

                if ((i + j) % 2 == 0)
                {
                    // Only draw black cells.
                    ofDrawRectangle(cellX, cellY, cellSize, cellSize);
                }

                if (i > 0 && j > 0)
                {
                    // Add normalized intersection points to the list.
                    float normX = ofMap(cellX, 0, fboProjection.getWidth(), 0, 1);
                    float normY = ofMap(cellY, 0, fboProjection.getHeight(), 0, 1);
                    currProjectorPoints.push_back(glm::vec2(normX, normY));
                }
            }
        }
    }
    fboProjection.end();
}

void ofApp::renderTestPoint(glm::vec2 projectedPoint)
{
    float pointSize = ofMap(sin(ofGetFrameNum() * 0.1), -1, 1, 3, 40);

    fboProjection.begin();
    {
        ofBackground(255);

        // Point is normalized, so it needs to be mapped to the projector size.
        float projX = ofMap(projectedPoint.x, 0, 1, 0, fboProjection.getWidth());
        float projY = ofMap(projectedPoint.y, 0, 1, 0, fboProjection.getHeight());

        ofSetColor(0, 255, 0);
        ofDrawCircle(projX, projY, pointSize);
    }
    fboProjection.end();
}

void ofApp::renderContours()
{
    fboProjection.begin();
    {
        // Clear white and draw black contours.
        ofClear(255, 0);
        ofSetColor(0);

        std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
        if (rsDevice)
        {
            for (int i = 0; i < contourFinder.size(); i++)
            {
                // Map contour using calibration and draw to main window
                ofBeginShape();
                std::vector<cv::Point> points = contourFinder.getContour(i);
                for (int j = 0; j < points.size(); j++)
                {
                    glm::vec3 worldPoint = rsDevice->getWorldPosition(points[j].x, points[j].y);
                    if (worldPoint.z > 0)
                    {
                        glm::vec2 projectedPoint = kpToolkit.getProjectedPoint(worldPoint);
                        if (ofInRange(projectedPoint.x, 0, 1) && ofInRange(projectedPoint.y, 0, 1))
                        {
                            if (flipX)
                            {
                                projectedPoint.x = 1.0 - projectedPoint.x;
                            }
                            if (flipY)
                            {
                                projectedPoint.y = 1.0 - projectedPoint.y;
                            }
                            ofVertex(projectedPoint.x * fboProjection.getWidth(), projectedPoint.y * fboProjection.getHeight());
                            ofLog() << "Adding world " << worldPoint << " // point " << projectedPoint << " // proj " << (projectedPoint.x * fboProjection.getWidth()) << ", " << (projectedPoint.y * fboProjection.getHeight());
                        }
                    }
                }
                ofEndShape();
            }
        }
    }
    fboProjection.end();
}

void ofApp::addPointPairs()
{
    std::shared_ptr<ofxRealSense2::Device> rsDevice = rsContext.getDevice(0);
    if (!rsDevice)
    {
        ofLogError(__FUNCTION__) << "No RealSense detected!";
        return;
    }

    // Count the number of world points.
    int numDepthPoints = 0;
    for (int i = 0; i < cvPoints.size(); i++)
    {
        glm::vec3 worldPoint = rsDevice->getWorldPosition(cvPoints[i].x, cvPoints[i].y);
        if (worldPoint.z > 0)
        {
            numDepthPoints++;
        }
    }

    int chessboardTotal = (CHESSBOARD_COLS - 1) * (CHESSBOARD_ROWS - 1);
    if (numDepthPoints != chessboardTotal)
    {
        ofLogError(__FUNCTION__) << "Only found " << numDepthPoints << " / " << chessboardTotal << " points!";
        return;
    }

    // Found all chessboard points in the world, add both sets to the lists we will use for calibration.
    for (int i = 0; i < cvPoints.size(); i++)
    {
        glm::vec3 worldPoint = rsDevice->getWorldPosition(cvPoints[i].x, cvPoints[i].y);
        pairsWorld.push_back(worldPoint);
        pairsProjector.push_back(currProjectorPoints[i]);
        ofLogNotice(__FUNCTION__) << "Adding pair i: " << worldPoint << " => " << currProjectorPoints[i];
    }

    ofLogNotice(__FUNCTION__) << "Added " << chessboardTotal << " point-pairs.";
}

void ofApp::calibrateSpaces()
{
    kpToolkit.calibrate(pairsWorld, pairsProjector);
    appMode = 1;

    pairsWorld.clear();
    pairsProjector.clear();
}

void ofApp::saveCalibration()
{
    if (kpToolkit.saveCalibration("calibration.json"))
    {
        ofLogNotice(__FUNCTION__) << "Calibration saved!";
    }
}

void ofApp::loadCalibration()
{
    if (kpToolkit.loadCalibration("calibration.json"))
    {
        ofLogNotice(__FUNCTION__) << "Calibration loaded!";
        appMode = 2;
    }
}
```
