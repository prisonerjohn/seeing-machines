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
    parent: "class-10"
    identifier: "mapping"
weight: 1020
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

Next, we will add an interface for mapping feature points from one image to the next.

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

## Combining Cameras

We will sometimes want to track a space that is larger than what a single camera can cover. We can use many sensors, but in order to combine their data correctly, we need to know what part of the space each is actually covering. While we could try to wing it by translating and rotating each point cloud, we will get much better results if we can get an accurate pose for each camera.

### Affine Transform

An [affine transform](https://mathworld.wolfram.com/AffineTransformation.html) is a transformation that preserves the relationships (sizes and distances) between points, lines, shapes. Because our cameras are both representing the same space without distorting it, the mapping from one to another is an affine transform.

To find the relationship from one sensor to the other, we will find a set of feature points that are visible in both viewports, and input those into a formula that will figure out the transformation that can convert each point from image A to image B. That transformation will be the same for the camera A pose to camera B pose!

We need to make sure there is some overlap between areas each sensor covers, as that is where we will find our feature points.

* One common way to get feature points is to use a flashlight or a reflector, and track the brightest blob in the images.
* Another common technique, which we will use now, is to track the corners of a chessboard.

{{< image src="https://docs.opencv.org/3.4/homography_perspective_correction_chessboard_warp.jpg" alt="Basic concepts of the homography explained with code" caption="*Basic concepts of the homography explained with code*" width="600px" >}}

### RealSense Data

The following example uses two RealSense cameras, but we could use any cameras we want, and mix and match.

First, we will write an app that displays the depth, color, and point cloud of each connected RealSense.

* The point clouds are drawn in the same area of the window, overlapping each other.
* Each point cloud is wrapped inside an `ofPushMatrix()` / `ofPopMatrix()` pair, so that we can apply separate transformations to each.
* The `debugPointClouds` flag allows us to draw each point cloud in a different color, to easily identify them.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxCv.h"
#include "ofxGui.h"
#include "ofxRealSense2.h"

class ofApp
  : public ofBaseApp 
{
public:
  void setup();
  void exit();

  void update();
  void draw();

  void keyPressed(int key);

  ofxRealSense2::Context context;
  ofEventListeners eventListeners;

  ofEasyCam cam;

  ofParameter<bool> debugPointClouds;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup() 
{
  ofDisableArbTex();

  eventListeners.push(context.deviceAddedEvent.newListener([&](std::string serialNumber) 
  {
    ofLogNotice(__FUNCTION__) << "Starting device " << serialNumber;
    
    auto device = context.getDevice(serialNumber);
    device->enableDepth();
    device->enableColor();
    device->enablePoints();
    device->startPipeline();
  }));

  try
  {
    context.setup(false);
  } 
  catch (std::exception & e)
  {
    ofLogFatalError(__FUNCTION__) << e.what();
  }

  debugPointClouds.set("Debug Point Clouds", false);

  guiPanel.setup("Calibrate Cams", "settings.json");
  guiPanel.add(debugPointClouds);
}

void ofApp::exit() 
{
  context.clear();
}

void ofApp::update() 
{
  context.update();
}

void ofApp::draw() 
{
  ofBackground(0);

  int count = MIN(this->context.getDevices().size(), 2);

  for (int i = 0; i < count; ++i)
  {
    auto device = context.getDevice(i);

    int x = 640 * i;
    device->getDepthTex().draw(x, 0);

    colorImg[i].draw(x, 360);
  }

  cam.begin(ofRectangle(1280, 0, 720, 720));
  ofEnableDepthTest();
  ofPushMatrix();
  ofScale(100);
  ofRotateXDeg(180);
  {
    // Draw device 0 transformed.
    ofPushMatrix();
    {
      ofDrawAxis(1.0);

      auto device0 = context.getDevice(0);

      if (debugPointClouds)
      {
        ofSetColor(255, 0, 0);
        device0->getPointsMesh().draw();
      }
      else
      {
        ofSetColor(255);
        device0->getColorTex().bind();
        device0->getPointsMesh().draw();
        device0->getColorTex().unbind();
      }
    }
    ofPopMatrix();

    // Draw device 1 at the origin.
    ofPushMatrix();
    {
      ofDrawAxis(1.0);
      ofSetColor(0, 255, 0);

      auto device1 = context.getDevice(1);

      if (debugPointClouds) 
      {
        ofSetColor(0, 255, 0);
        device1->getPointsMesh().draw();
      } 
      else
      {
        ofSetColor(255);
        device1->getColorTex().bind();
        device1->getPointsMesh().draw();
        device1->getColorTex().unbind();
      }
    }
    ofPopMatrix();
  }
  ofPopMatrix();
  ofDisableDepthTest();
  cam.end();

  ofSetColor(255);

  ofDrawBitmapString(ofToString(ofGetFrameRate()), 10, 10);

  guiPanel.draw();
}

void ofApp::keyPressed(int key) 
{

}
```

### OpenCV Chessboard Detection

We will use a printed chessboard and hold it in front of the depth sensors.

* The chessboard is detected using the sensor's color cameras. Both images are passed to OpenCV to detect the pattern and give us the coordinates of each corner (intersection point).

This process takes a few steps:

* [`cv::findChessboardCorners()`](https://docs.opencv.org/3.4/d9/d0c/group__calib3d.html#ga93efa9b0aa890de240ca32b11253dd4a) finds the chessboard pattern in an image. Make sure you pass the correct number of corners to look for as the `patternSize` parameter.
* [cv::cornerSubPix()`](https://docs.opencv.org/3.4/dd/d1a/group__imgproc__feature.html#ga354e0d7c86d0d9da75de9b9701a9a87e) refines the found corners to give better results.
* [`cv::drawChessboardCorners()`](https://docs.opencv.org/3.4/d9/d0c/group__calib3d.html#ga6a10b0bb120c4907e5eabbcd22319022) draws the found corners into the image. While this step is not necessary, it is very useful for debugging!

{{< image src="chessboards.png" alt="Chessboards" caption="*Chessboards*" width="600px" >}}

Because we need corresponding points (or *point pairs*), we will only consider frames where the chessboard was found in both images.

* We will save coordinate pairs in vectors by pressing a button. We will also add a button to clear the vectors and start over.
* The `imgPoints` vectors will hold the chessboard corner coordinates in image space (2D) for the current frame only. This will be updated every frame.
* The `worldPoints` vectors will hold the world coordinates of the chessboard corners (3D) for all saved frames. This will only be updated when `savePoints` is enabled.
* The `imgPoints` coordinates are fed to the sensor's world coordinate mapper to extract corresponding 3D world points.
* To get the world points from a coordinate, we will need to make sure our depth and color images are aligned / registered using the corresponding SDK call. For `ofxRealSense2`, this means setting each device's `alignMode` to `ofxRealSense2::Device::Align::Color` or `ofxRealSense2::Device::Align::Depth`.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxCv.h"
#include "ofxGui.h"
#include "ofxRealSense2.h"

class ofApp
  : public ofBaseApp 
{
public:
  void setup();
  void exit();

  void update();
  void draw();

  void keyPressed(int key);

  ofxRealSense2::Context context;
  ofEventListeners eventListeners;

  cv::Mat grayMat[2];
  cv::Mat colorMat[2];
  ofImage colorImg[2];

  std::vector<cv::Point2f> imgPoints[2];
  std::vector<glm::vec3> worldPoints[2];

  ofEasyCam cam;

  ofParameter<bool> savePoints;
  ofParameter<bool> clearPoints;
  ofParameter<bool> debugPointClouds;
  ofxLabel statusPoints;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup() 
{
  ofDisableArbTex();

  guiPanel.setup("settings.xml");

  eventListeners.push(context.deviceAddedEvent.newListener([&](std::string serialNumber) 
  {
    ofLogNotice(__FUNCTION__) << "Starting device " << serialNumber;
    
    auto device = context.getDevice(serialNumber);
    device->enableDepth();
    device->enableColor();
    device->enablePoints();
    device->startPipeline();
  }));

  guiPanel.setup("Calibrate Cams", "settings.json");

  try
  {
    context.setup(false);
  } 
  catch (std::exception & e)
  {
    ofLogFatalError(__FUNCTION__) << e.what();
  }
}

void ofApp::exit() 
{
  context.clear();
}

void ofApp::update() 
{
  context.update();

  // Number of corners in the grid, i.e. number of cols - 1 and number of rows - 1.
  cv::Size patternSize = cv::Size(10, 7);
  int count = MIN(this->context.getDevices().size(), 2);
  for (int i = 0; i < count; ++i) 
  {
    auto device = context.getDevice(i);

    // Align the frames to the color viewport.
    device->alignMode = ofxRealSense2::Device::Align::Color;

    // Find chessboard pattern in image.
    imgPoints[i].clear();
    colorMat[i] = ofxCv::toCv(device->getColorPix());
    int chessFlags = cv::CALIB_CB_ADAPTIVE_THRESH + cv::CALIB_CB_FAST_CHECK;
    bool foundChessboard = cv::findChessboardCorners(colorMat[i], patternSize, imgPoints[i], chessFlags);
    if (foundChessboard) 
    {
      // Refine the corners.
      // cv::cornerSubPix() requires a grayscale image, so we need to convert our color image first.
      cv::cvtColor(colorMat[i], grayMat[i], CV_RGB2GRAY);
      cv::cornerSubPix(grayMat[i], imgPoints[i], cv::Size(11, 11), cv::Size(-1, -1),
        cv::TermCriteria(CV_TERMCRIT_EPS + CV_TERMCRIT_ITER, 30, 0.1));

      // Draw the corners over the color image for review.
      cv::drawChessboardCorners(colorMat[i], patternSize, cv::Mat(imgPoints[i]), foundChessboard);
    }

    ofxCv::toOf(colorMat[i], colorImg[i]);
    colorImg[i].update();
  }

  if (clearPoints)
  {
    worldPoints[0].clear();
    worldPoints[1].clear();

    clearPoints = false;
  }

  if (savePoints)
  {
    // Only save points if the same number was found in both images.
    if (count == 2 &&
      !imgPoints[0].empty() &&
      imgPoints[0].size() == imgPoints[1].size())
    {
      auto device0 = context.getDevice(0);
      auto device1 = context.getDevice(1);

      int added = 0;
      for (int i = 0; i < imgPoints[0].size(); ++i)
      {
        glm::vec3 point0 = device0->getWorldPosition(imgPoints[0][i].x, imgPoints[0][i].y);
        glm::vec3 point1 = device1->getWorldPosition(imgPoints[1][i].x, imgPoints[1][i].y);
        
        // Only add if both points are valid.
        if (point0.z != 0 && point1.z != 0)
        {
          worldPoints[0].push_back(glm::vec3(point0.x, point0.y, point0.z));
          worldPoints[1].push_back(glm::vec3(point1.x, point1.y, point1.z));

          ++added;
        }
      }
      ofLogNotice(__FUNCTION__) << "Added " << added << " point pairs";
    }
    else
    {
      ofLogWarning(__FUNCTION__) << "Found points count mismatch! " << imgPoints[0].size() << " vs " << imgPoints[1].size();
    }

    savePoints = false;
  }

  statusPoints = ofToString(worldPoints[0].size()) + " Point Pairs";
}

void ofApp::draw() 
{
  ofBackground(0);

  int count = MIN(this->context.getDevices().size(), 2);

  for (int i = 0; i < count; ++i)
  {
    auto device = context.getDevice(i);

    int x = 640 * i;
    device->getDepthTex().draw(x, 0);

    colorImg[i].draw(x, 360);
  }

  cam.begin(ofRectangle(1280, 0, 720, 720));
  ofEnableDepthTest();
  ofPushMatrix();
  ofScale(100);
  ofRotateXDeg(180);
  {

    // Draw device 0 transformed.
    ofPushMatrix();
    {
      ofDrawAxis(1.0);

      auto device0 = context.getDevice(0);

      if (debugPointClouds)
      {
        ofSetColor(255, 0, 0);
        device0->getPointsMesh().draw();
      }
      else
      {
        ofSetColor(255);
        device0->getColorTex().bind();
        device0->getPointsMesh().draw();
        device0->getColorTex().unbind();
      }
    }
    ofPopMatrix();

    // Draw device 1 at the origin.
    ofPushMatrix();
    {
      ofDrawAxis(1.0);
      ofSetColor(0, 255, 0);

      auto device1 = context.getDevice(1);

      if (debugPointClouds) 
      {
        ofSetColor(0, 255, 0);
        device1->getPointsMesh().draw();
      } 
      else
      {
        ofSetColor(255);
        device1->getColorTex().bind();
        device1->getPointsMesh().draw();
        device1->getColorTex().unbind();
      }
    }
    ofPopMatrix();
  }
  ofPopMatrix();
  ofDisableDepthTest();
  cam.end();

  ofSetColor(255);

  ofDrawBitmapString(ofToString(ofGetFrameRate()), 10, 10);

  guiPanel.draw();
}

void ofApp::keyPressed(int key) 
{
  if (key == ' ')
  {
    savePoints = true;
  }
}
```

### OpenCV Calibration

Finally, we can calibrate the cameras by passing both sets of world points to the [`cv::estimateAffine3D()`](https://docs.opencv.org/3.4/d9/d0c/group__calib3d.html#ga27865b1d26bac9ce91efaee83e94d4dd) OpenCV function.

* We will write our own `estimateAffine3D()` method in our `ofApp` that will convert the input data from OF to CV and the output data from CV back to OF.
* Although the `ofxCv` addon includes a wrapper for `cv::estimateAffine3D()`, it is using a different version of the OpenCV function from what we want, which is why we are writing our own wrapper.
* This function will return a transformation matrix. We can apply it to our second camera's point cloud using `ofMultMatrix()`.

The final code is below:

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxCv.h"
#include "ofxGui.h"
#include "ofxRealSense2.h"

class ofApp
  : public ofBaseApp 
{
public:
  void setup();
  void exit();

  void update();
  void draw();

  void keyPressed(int key);

  glm::mat4 estimateAffine3D(float accuracy = 0.99f);

  ofxRealSense2::Context context;
  ofEventListeners eventListeners;

  cv::Mat grayMat[2];
  cv::Mat colorMat[2];
  ofImage colorImg[2];

  std::vector<cv::Point2f> imgPoints[2];
  std::vector<glm::vec3> worldPoints[2];

  glm::mat4x4 estimatedTransform;

  ofEasyCam cam;

  ofParameter<bool> savePoints;
  ofParameter<bool> clearPoints;
  ofParameter<bool> calibrateSensors;
  ofParameter<bool> debugPointClouds;
  ofxLabel statusPoints;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup() 
{
  ofDisableArbTex();

  guiPanel.setup("settings.xml");

  eventListeners.push(context.deviceAddedEvent.newListener([&](std::string serialNumber) 
  {
    ofLogNotice(__FUNCTION__) << "Starting device " << serialNumber;
    
    auto device = context.getDevice(serialNumber);
    device->enableDepth();
    device->enableColor();
    device->enablePoints();
    device->startPipeline();
  }));

  guiPanel.setup("Calibrate Cams", "settings.json");

  try
  {
    context.setup(false);
  } 
  catch (std::exception & e)
  {
    ofLogFatalError(__FUNCTION__) << e.what();
  }
}

void ofApp::exit() 
{
  context.clear();
}

void ofApp::update() 
{
  context.update();

  // Number of corners in the grid, i.e. number of cols - 1 and number of rows - 1.
  cv::Size patternSize = cv::Size(10, 7);
  int count = MIN(this->context.getDevices().size(), 2);
  for (int i = 0; i < count; ++i) 
  {
    auto device = context.getDevice(i);

    // Align the frames to the color viewport.
    device->alignMode = ofxRealSense2::Device::Align::Color;

    // Find chessboard pattern in image.
    imgPoints[i].clear();
    colorMat[i] = ofxCv::toCv(device->getColorPix());
    int chessFlags = cv::CALIB_CB_ADAPTIVE_THRESH + cv::CALIB_CB_FAST_CHECK;
    bool foundChessboard = cv::findChessboardCorners(colorMat[i], patternSize, imgPoints[i], chessFlags);
    if (foundChessboard) 
    {
      // Refine the corners.
      cv::cvtColor(colorMat[i], grayMat[i], CV_RGB2GRAY);
      cv::cornerSubPix(grayMat[i], imgPoints[i], cv::Size(11, 11), cv::Size(-1, -1),
        cv::TermCriteria(CV_TERMCRIT_EPS + CV_TERMCRIT_ITER, 30, 0.1));

      // Draw the corners over the color image for review.
      cv::drawChessboardCorners(colorMat[i], patternSize, cv::Mat(imgPoints[i]), foundChessboard);
    }

    ofxCv::toOf(colorMat[i], colorImg[i]);
    colorImg[i].update();
  }

  if (clearPoints)
  {
    worldPoints[0].clear();
    worldPoints[1].clear();

    clearPoints = false;
  }

  if (savePoints)
  {
    // Only save points if the same number was found in both images.
    if (count == 2 &&
      !imgPoints[0].empty() &&
      imgPoints[0].size() == imgPoints[1].size())
    {
      auto device0 = context.getDevice(0);
      auto device1 = context.getDevice(1);

      int added = 0;
      for (int i = 0; i < imgPoints[0].size(); ++i)
      {
        glm::vec3 point0 = device0->getWorldPosition(imgPoints[0][i].x, imgPoints[0][i].y);
        glm::vec3 point1 = device1->getWorldPosition(imgPoints[1][i].x, imgPoints[1][i].y);
        
        // Only add if both points are valid.
        if (point0.z != 0 && point1.z != 0)
        {
          worldPoints[0].push_back(glm::vec3(point0.x, point0.y, point0.z));
          worldPoints[1].push_back(glm::vec3(point1.x, point1.y, point1.z));

          ++added;
        }
      }
      ofLogNotice(__FUNCTION__) << "Added " << added << " point pairs";
    }
    else
    {
      ofLogWarning(__FUNCTION__) << "Found points count mismatch! " << imgPoints[0].size() << " vs " << imgPoints[1].size();
    }

    savePoints = false;
  }

  if (calibrateSensors)
  {
    // We are using our own estimateAffine3D because ofxCv does not use the version we need from OpenCV.
    estimatedTransform = estimateAffine3D();

    calibrateSensors = false;
  }

  statusPoints = ofToString(worldPoints[0].size()) + " Point Pairs";
}

void ofApp::draw() 
{
  ofBackground(0);

  int count = MIN(this->context.getDevices().size(), 2);

  for (int i = 0; i < count; ++i)
  {
    auto device = context.getDevice(i);

    int x = 640 * i;
    device->getDepthTex().draw(x, 0);

    colorImg[i].draw(x, 360);
  }

  cam.begin(ofRectangle(1280, 0, 720, 720));
  ofEnableDepthTest();
  ofPushMatrix();
  ofScale(100);
  ofRotateXDeg(180);
  {

    // Draw device 0 transformed.
    ofPushMatrix();
    ofMultMatrix(estimatedTransform);
    {
      ofDrawAxis(1.0);

      auto device0 = context.getDevice(0);

      if (debugPointClouds)
      {
        ofSetColor(255, 0, 0);
        device0->getPointsMesh().draw();
      }
      else
      {
        ofSetColor(255);
        device0->getColorTex().bind();
        device0->getPointsMesh().draw();
        device0->getColorTex().unbind();
      }
    }
    ofPopMatrix();

    // Draw device 1 at the origin.
    ofPushMatrix();
    {
      ofDrawAxis(1.0);
      ofSetColor(0, 255, 0);

      auto device1 = context.getDevice(1);

      if (debugPointClouds) 
      {
        ofSetColor(0, 255, 0);
        device1->getPointsMesh().draw();
      } 
      else
      {
        ofSetColor(255);
        device1->getColorTex().bind();
        device1->getPointsMesh().draw();
        device1->getColorTex().unbind();
      }
    }
    ofPopMatrix();
  }
  ofPopMatrix();
  ofDisableDepthTest();
  cam.end();

  ofSetColor(255);

  ofDrawBitmapString(ofToString(ofGetFrameRate()), 10, 10);

  guiPanel.draw();
}

void ofApp::keyPressed(int key) 
{
  if (key == ' ')
  {
    savePoints = true;
  }
  else if (key == OF_KEY_RETURN)
  {
    calibrateSensors = true;
  }
}

glm::mat4 ofApp::estimateAffine3D(float accuracy) 
{
  cv::Mat srcMat(1, worldPoints[0].size(), CV_32FC3, worldPoints[0].data());
  cv::Mat dstMat(1, worldPoints[1].size(), CV_32FC3, worldPoints[1].data());

  cv::Mat affineMat = cv::estimateAffine3D(srcMat, dstMat);

  // Convert the transformation matrix from OpenCV format to OF (glm) format.
  auto affineMatPtr = affineMat.ptr<double>();
  glm::mat4 affine = glm::mat4(1.0f);
  auto affinePtr = glm::value_ptr(affine);
  for (int i = 0; i < 12; ++i)
  {
    affinePtr[i] = affineMatPtr[i];
  }
  affine = glm::transpose(affine);

  return affine;
}
```

{{< image src="calibrated-cameras.png" alt="Merged Cameras" caption="*Calibrated Cameras*" width="600px" >}}

## World to Projector Mapping

Another common mapping operation is to map the 3D space we are projecting onto back into the projector image.

<figure style="width:600px;height:420px;display:block;margin:0 auto;">
  <iframe width="600" height="375" src="https://www.youtube.com/embed/CE1B7tdGCw0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
  <figcaption><i>UCLA's Augmented Reality Sandbox</i></figcaption>
</figure>

The idea is similar to what we have seen so far:

* We will collect pairs of corresponding points from both spaces. 2D points from the projector, and 3D points from the sensor covering the space.
* We will pass these point pairs to a solver function, which will give us a transformation we can apply to other points in the same 3D space, and have them *project* to an appropriate position on the 2D screen.

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

[ofxKinectProjectorToolkit](https://github.com/prisonerjohn/ofxKinectProjectorToolkit/tree/develop) is one of the many addons available for OF to create a correspondence between the 3D world and a 2D projector.

* This addon was originally written by Gene Kogan but it's a little out of date. The link above is my updated version which should get you started faster.
* As the name implies, this addon is meant to work with the Kinect sensor. However, the calibration functions are sensor-agnostic. You just need to provide point pairs and it does the rest.
* That being said, some sensors will provide better data than others. For example, a stereo sensor like the RealSense will have very noisy depth data which will need to be filtered before it can be useful.

We will also use a chessboard pattern and the [`cv::findChessboardCorners()`](https://docs.opencv.org/3.4/d9/d0c/group__calib3d.html#ga93efa9b0aa890de240ca32b11253dd4a) function. However, because one of our spaces is the projector and the other (the camera) is capturing this projector, we will render a digital chessboard out to the screen.

* Because we are rendering the chessboard corners out of the projector, we already know their 2D position on screen, in projector space.
* The chessboard is detected using the depth sensor's color camera. The tracked points are then fed to the sensor's world coordinate mapper to extract corresponding 3D world points.
* The two sets of points are then fed to a solver, which determines the transformation matrix from one space to the other.
* `ofxKinectProjectorToolkit` uses [dlib](http://dlib.net/) for calibration, which is another commonly used image processing library.
* If we were to use OpenCV, we would probably use the [`cv::calibrateCamera()`](https://docs.opencv.org/2.4/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html#calibratecamera) function.

The following is a simplified version of the addon's calibration example:

```cpp
// main.cpp
#include "ofApp.h"
#include "ofMain.h"

int main()
{
  ofGLFWWindowSettings settings;

  settings.setSize(1280, 900);
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
  ofAddListener(secondWindow->events().draw, mainApp.get(), &ofApp::drawSecondWindow);

  ofRunApp(mainWindow, mainApp);
  ofRunMainLoop();
}
```

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxCv.h"
#include "ofxKinect.h"
#include "ofxKinectProjectorToolkit.h"

// This must match the display resolution of the projector.
#define PROJECTOR_RESOLUTION_X 1920
#define PROJECTOR_RESOLUTION_Y 1080

class ofApp : public ofBaseApp 
{
public:
  void setup();
  void update();
  void draw();
  void drawSecondWindow(ofEventArgs & args);

  void keyPressed(int key);
  void mousePressed(int x, int y, int button);

  void drawChessboard(int x, int y, int chessboardSize);
  void drawTestingPoint(glm::vec2 projectedPoint);
  void addPointPair();

  ofxKinect kinect;

  ofxKinectProjectorToolkit kpt;

  ofFbo fboChessboard;
  ofImage colorImg;
  cv::Mat colorMat;

  vector<glm::vec2> currentProjectorPoints;
  vector<cv::Point2f> cvPoints;
  vector<glm::vec3> pairsKinect;
  vector<glm::vec2> pairsProjector;

  glm::vec2 testPoint;

  int chessboardSize;
  int chessboardX;
  int chessboardY;
  bool testing;
  bool saved;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup() 
{
  chessboardSize = 300;
  chessboardX = 5;
  chessboardY = 4;

  kinect.setRegistration(true);
  kinect.init();
  kinect.open();

  fboChessboard.allocate(PROJECTOR_RESOLUTION_X, PROJECTOR_RESOLUTION_Y, GL_RGBA);

  testing = false;
}

void ofApp::drawChessboard(int x, int y, int chessboardSize) 
{
  float w = chessboardSize / chessboardX;
  float h = chessboardSize / chessboardY;

  currentProjectorPoints.clear();

  // Render a chessboard in an FBO.
  fboChessboard.begin();
  {
    ofClear(255, 0);
    ofSetColor(0);
    ofTranslate(x, y);
    for (int j = 0; j < chessboardY; j++) 
    {
      for (int i = 0; i < chessboardX; i++) 
      {
        int x0 = ofMap(i, 0, chessboardX, 0, chessboardSize);
        int y0 = ofMap(j, 0, chessboardY, 0, chessboardSize);
        if (j > 0 && i > 0) 
        {
          // Save the 2D corner point.
          currentProjectorPoints.push_back(ofVec2f(
            ofMap(x + x0, 0, fboChessboard.getWidth(), 0, 1),
            ofMap(y + y0, 0, fboChessboard.getHeight(), 0, 1)));
        }
        if ((i + j) % 2 == 0) 
        {
          // Draw a black rectangle every other tile.
          ofDrawRectangle(x0, y0, w, h);
        }
      }
    }
    ofSetColor(255);
  }
  fboChessboard.end();
}

void ofApp::drawTestingPoint(glm::vec2 projectedPoint) 
{
  // Draw the projected testing point in the FBO.
  float ptSize = ofMap(sin(ofGetFrameNum() * 0.1), -1, 1, 3, 40);
  fboChessboard.begin();
  {
    ofBackground(255);
    ofSetColor(0, 255, 0);
    ofCircle(
      ofMap(projectedPoint.x, 0, 1, 0, fboChessboard.getWidth()),
      ofMap(projectedPoint.y, 0, 1, 0, fboChessboard.getHeight()),
      ptSize);
    ofSetColor(255);
  }
  fboChessboard.end();
}

void ofApp::addPointPair() 
{
  // Find corresponding 3D world points for each 2D chessboard corner.

  // Count the number of found points...
  int nDepthPoints = 0;
  for (int i = 0; i < cvPoints.size(); ++i) 
  {
    glm::vec3 worldPoint = kinect.getWorldCoordinateAt(cvPoints[i].x, cvPoints[i].y);
    if (worldPoint.z > 0) ++nDepthPoints;
  }
  // ...and add them only if all corners are found.
  if (nDepthPoints == (chessboardX - 1) * (chessboardY - 1)) 
  {
    for (int i = 0; i < cvPoints.size(); ++i) 
    {
      glm::vec3 worldPoint = kinect.getWorldCoordinateAt(cvPoints[i].x, cvPoints[i].y);

      ofLogNotice(__FUNCTION__) << "Point pair " << currentProjectorPoints[i] << " => " << worldPoint;

      pairsKinect.push_back(worldPoint);
      pairsProjector.push_back(currentProjectorPoints[i]);
    }

    ofLogNotice(__FUNCTION__) << "Added " << ((chessboardX - 1) * (chessboardY - 1)) << " points pairs.";
  } 
  else
  {
    ofLogWarning(__FUNCTION__) << "Points not added because not all chessboard points' depth known. Try re-positionining.";
  }
}

void ofApp::update() 
{
  kinect.update();

  if (kinect.isFrameNew())
  {
    colorImg.setFromPixels(kinect.getPixels());

    if (testing) 
    {
      // Calculate the projected value of the testing point and render it.
      glm::vec2 t = glm::vec2(MIN(kinect.getWidth() - 1, testPoint.x), MIN(kinect.getHeight() - 1, testPoint.y));
      ofVec3f worldPoint = kinect.getWorldCoordinateAt(t.x, t.y);
      ofVec2f projectedPoint = kpt.getProjectedPoint(worldPoint);
      drawTestingPoint(projectedPoint);
    } 
    else
    {
      // Draw a chessboard on the projector...
      drawChessboard(ofGetMouseX(), ofGetMouseY(), chessboardSize);

      // ...and use OpenCV to find it in the Kinect color image.
      colorMat = ofxCv::toCv(colorImg);
      cv::Size patternSize = cv::Size(chessboardX - 1, chessboardY - 1);
      int chessFlags = cv::CALIB_CB_ADAPTIVE_THRESH + cv::CALIB_CB_FAST_CHECK;
      bool foundChessboard = cv::findChessboardCorners(colorMat, patternSize, cvPoints, chessFlags);
      if (foundChessboard) 
      {
        cv::Mat gray;
        cv::cvtColor(colorMat, gray, CV_RGB2GRAY);
        cv::cornerSubPix(gray, cvPoints, cv::Size(11, 11), cv::Size(-1, -1),
          cv::TermCriteria(CV_TERMCRIT_EPS + CV_TERMCRIT_ITER, 30, 0.1));
        cv::drawChessboardCorners(colorMat, patternSize, cv::Mat(cvPoints), foundChessboard);

        colorImg.update();
      }
    }
  }
}

void ofApp::draw() 
{
  colorImg.draw(0, 0);
  kinect.drawDepth(0, 490, 320, 240);

  std::ostringstream oss;

  ofSetColor(0);
  if (testing) 
  {
    oss << "Click on the image to test a point in the RGB image." << std::endl
      << "The projector should place a green dot on the corresponding point." << std::endl
      << "Press the 's' key to save the calibration." << std::endl;
    if (saved) 
    {
      oss << "Calibration saved." << std::endl;
    }

    ofSetColor(255, 0, 0);
    float ptSize = ofMap(cos(ofGetFrameNum() * 0.1), -1, 1, 3, 40);
    ofCircle(testPoint.x, testPoint.y, ptSize);
  } 
  else 
  {
    oss << "Position the chessboard using the mouse." << std::endl
      << "Adjust the size of the chessboard using the 'q' and 'w' keys." << std::endl
      << "Press the spacebar to save a set of point pairs." << std::endl
      << "Press the 'c' key to calibrate." << std::endl
      << pairsKinect.size() << " point pairs collected." << std::endl;
  }

  ofSetColor(255);
  ofDrawBitmapString(oss.str(), 532, 532);
}

void ofApp::drawSecondWindow(ofEventArgs & args) 
{
  ofSetColor(ofColor::white);
  fboChessboard.draw(0, 0);
}

void ofApp::keyPressed(int key) 
{
  if (key == ' ') 
  {
    addPointPair();
  } 
  else if (key == 'q')
  {
    chessboardSize -= 20;
  } 
  else if (key == 'w') 
  {
    chessboardSize += 20;
  }
  else if (key == 't')
  {
    testing = !testing;
  }
  else if (key == 'c')
  {
    kpt.calibrate(pairsKinect, pairsProjector);
    testing = true;
  } 
  else if (key == 's') 
  {
    kpt.saveCalibration("calibration.json");
    saved = true;
  } 
  else if (key == 'l')
  {
    kpt.loadCalibration("calibration.json");
    testing = true;
  }
}

void ofApp::mousePressed(int x, int y, int button) 
{
  if (testing) 
  {
    // Move the testing point to the mouse position.
    testPoint = glm::vec2(MIN(x, kinect.getWidth() - 1), MIN(y, kinect.getHeight() - 1));
  }
}
```

For better results, try tracking the chessboard at different positions, sizes, and depths. This might mean temporarily adding a large projection surface in front of the wall.

### Projected Blobs

The following is a simplified version of the addon's contours example, which tracks blobs using depth thresholding and reprojects a color directly on them.

* The `calibration.json` file is copied from the previous project's `bin/data` folder so that it can be used in this example.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxCv.h"
#include "ofxGui.h"
#include "ofxKinect.h"
#include "ofxKinectProjectorToolkit.h"

// this must match the display resolution of your projector
#define PROJECTOR_RESOLUTION_X 1920
#define PROJECTOR_RESOLUTION_Y 1080

class ofApp : public ofBaseApp 
{
public:
  void setup();
  void update();
  void draw();
  void drawSecondWindow(ofEventArgs & args);

  void keyPressed(int key);

  ofxKinect kinect;

  ofFloatPixels thresholdNear;
  ofFloatPixels thresholdFar;
  ofFloatPixels thresholdResult;

  ofImage thresholdImg;

  ofxCv::ContourFinder contourFinder;

  ofxKinectProjectorToolkit kpt;
  ofImage colorImg;
  cv::Mat colorMat;

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
  kinect.setRegistration(true);
  kinect.init();
  kinect.open();

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

  kpt.loadCalibration("calibration.json");
}

void ofApp::update() 
{
  kinect.update();

  if (kinect.isFrameNew())
  {
    // Threshold image with distance.
    ofFloatPixels depthFloatPixels = kinect.getRawDepthPixels();
    ofxCv::threshold(depthFloatPixels, thresholdNear, nearThreshold);
    ofxCv::threshold(depthFloatPixels, thresholdFar, farThreshold, true);
    ofxCv::bitwise_and(thresholdNear, thresholdFar, thresholdResult);
    thresholdImg.setFromPixels(thresholdResult);

    // Find contours.
    contourFinder.setMinAreaNorm(minArea);
    contourFinder.setMaxAreaNorm(maxArea);
    contourFinder.findContours(thresholdImg);
  }
}

void ofApp::draw() 
{
  kinect.draw(0, 0);
  contourFinder.draw();
  thresholdImg.draw(0, 454);

  guiPanel.draw();
}

void ofApp::drawSecondWindow(ofEventArgs & args) 
{
  ofSetColor(255, 0, 0);

  for (int i = 0; i < contourFinder.size(); ++i) 
  {
    std::vector<cv::Point> points = contourFinder.getContour(i);

    // Map contour from the world to the screen using the calibration transform.
    ofBeginShape();
    ofFill();
    ofSetColor(255, 0, 0);
    for (int j = 0; j < points.size(); ++j) 
    {
      glm::vec3 worldPoint = kinect.getWorldCoordinateAt(points[j].x, points[j].y);
      glm::vec2 projectedPoint = kpt.getProjectedPoint(worldPoint);
      ofVertex(PROJECTOR_RESOLUTION_X * projectedPoint.x, PROJECTOR_RESOLUTION_Y * projectedPoint.y);
    }
    ofEndShape();
  }
}

void ofApp::keyPressed(int key) 
{
  if (key == 'l')
  {
    kpt.loadCalibration("calibration.json");
  }
}
```
