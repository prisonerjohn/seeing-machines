---
title: "Machine Learning"
description: ""
lead: ""
date: 2022-11-27T12:04:16-05:00
lastmod: 2022-11-27T12:04:16-05:00
draft: false
images: []
menu:
  docs:
    parent: "class-12"
    identifier: "machine-learning"
weight: 1210
toc: true
---

Machine learning (ML) is a subset of artificial intelligence (AI) technologies. ML identifies patterns within training data and uses these patterns to make predictions on new data. One interesting fact about ML is that the learning happens indirectly instead of with explicitly defined algorithms. The more training and data a machine learning algorithm has, the more accurate it gets, similarly to how humans improve skills through practice.

This prediction algorithm is called a *model*.

Many steps are necessary to create a machine learning model:

1. Input data has to be collected (and cleaned).
1. A training program has to be developed.
1. A model gets trained with the input data.
1. The model is tested for correctness (and possibly adjusted and retrained).
1. Once the model is working, it can be used for predicting results using new data.

As you can imagine, this training process takes a lot of work and time. It is very resource intensive and can require special hardware. However, many pre-trained models are distributed for everyone to use, and a handful of frameworks are available to bring these models into our applications.

{{< alert context="info" icon="✌️" >}}
**ML Performance**

While many ML algorithms can run on all types of machines, they tend to be very resource intensive and will benefit from fast dedicated hardware. Models may come in CPU and GPU variants, where the GPU variant is orders of magnitude faster!

Most current generation phones even have a dedicated processor for running ML models, like [Apple Neural Engine](https://developer.apple.com/machine-learning/core-ml/) in iPhones and [Google Tensor](https://blog.google/products/pixel/introducing-google-tensor/) in Androids.
{{< /alert >}}

Models need to run on a *platform*. This is basically the interface that can format the input data correctly, send it to the model, and read out and interpret the prediction results.

Machine learning has a variety of applications like recommendation engines, route optimization, speech recognition, chat bots, etc. It can be particularly useful in the context of this class and computer vision for object detection.

## OpenCV

OpenCV includes a Deep Neural Network (DNN) module which makes it easy to load a model and use it in an OpenCV / openFrameworks based application. The model will also automatically benefit from the optimizations in OpenCV. An image classification example called `opencvImageClassification` is available in the `/path/to/OF/examples/computer_vision/` directory, which we can use as our reference.

### YOLOv5

The image classifier uses the [YOLOv5](https://ultralytics.com/yolov5) machine learning model for object detection. YOLOv5 is a popular pre-trained model used to identify objects in an image in realtime.

The model comes in [5 variants](https://docs.ultralytics.com/tutorials/train-custom-datasets/), each having a performance vs accuracy tradeoff. We will use the "nano" version as that is what ships with OF, and can be found in a file called `yolov5n.onnx` in the `bin/data` folder of the example. The folder also includes a `classes.txt` file, where each class ID is named on a separate line. So for example, class `0` represents a "person", class `1` a "bicycle", class `2` a "car", etc. These two files need to be copied over to the `bin/data` folder of any app we build using the YOLOv5 model.

We can update the example using `ofxCv`, as that is the interface we have been using for OpenCV in this class. We will also bring in the `yolo5ImageClassify.h` file into our new project and use it as-is to interpret the prediction results.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxCv.h"
#include "yolo5ImageClassify.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  ofVideoGrabber grabber;
  yolo5ImageClassify classify;
  vector<yolo5ImageClassify::Result> results;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  // Start video grabber.
  grabber.setup(640, 480);

  // Load the YOLOv5n model and the classes file.
  classify.setup("yolov5n.onnx", "classes.txt", false);
}

void ofApp::update()
{
  grabber.update();
  if ( grabber.isFrameNew())
  {
    // Get the grabber pixels reference.
    ofPixels& grabberPix = grabber.getPixels();
    // Convert the pixels to an OpenCV Mat.
    cv::Mat grabberMat = ofxCv::toCv(grabberPix);
    // Run the classifier model.
    results = classify.classifyFrame(grabberMat);
  }
}

void ofApp::draw()
{
  ofSetColor(255);
  grabber.draw(0, 0);

  // Draw the detected objects on top of the video.
  ofNoFill();
  ofSetColor(255, 0, 255);
  for (int i = 0; i < results.size(); i++)
  {
    yolo5ImageClassify::Result& result = results[i];
    ofDrawRectangle(result.rect);
    ofDrawBitmapStringHighlight(result.label, result.rect.getTopLeft());
  }
}
```

{{< image src="cv-yolo.png" alt="OpenCV YOLOv5" width="600px" >}}

## ofxTensorFlow2

[TensorFlow](https://www.tensorflow.org/) is a machine learning platform developed by Google. It is very popular because it is open-source, comprehensive, and can be used in many programming languages.

[ofxTensorFlow2](https://github.com/zkmkarlsruhe/ofxTensorFlow2) is a third-party addon for openFrameworks to use the TensorFlow framework. The addon includes many examples including computer vision models like Pose Estimation and Video Matting as well as other models like Style Transfer.

{{< image src="https://github.com/zkmkarlsruhe/ofxTensorFlow2/raw/main/media/movenet.gif" alt="ofxTensorFlow2 Pose Estimation" caption="*ofxTensorFlow2 Pose Estimation*" width="600px" >}}

{{< image src="https://github.com/zkmkarlsruhe/ofxTensorFlow2/raw/main/media/style_transfer.gif" alt="ofxTensorFlow2 Style Transfer" caption="*ofxTensorFlow2 Style Transfer*" width="600px" >}}

Installing the addon requires a few more steps after downloading the addon ZIP, as outlined in the [README](https://github.com/zkmkarlsruhe/ofxTensorFlow2/blob/main/README.md). The additional `cppflow` and `tensorflow` libraries as well as the models used in the examples need to be downloaded separately. This is a bit easier on Mac, as the download scripts are formatted for Mac/Linux.

For simplicity's sake, I have the full addon available here for download on [Mac (CPU only)](https://drive.google.com/file/d/1GAbbyTHJzqOKhpu_r029ayCmFE0tREPg/view?usp=sharing), [Windows (CPU)](https://drive.google.com/file/d/1hUR2sRGyDKsc_L7Sk3D45y2TdquBvM0O/view?usp=sharing), and [Windows (GPU)](https://drive.google.com/file/d/18m4weedXDQqr7UZzPQohu_5XMg_WJ2zX/view?usp=sharing).

Note that on macOS, there are a couple of extra steps to perform in Xcode.

1. Change the C++ language version to `c++14`.
  {{< image src="xcode-cpp14.jpg" alt="Xcode C++14" align="center" >}}
1. Run a custom linking script in the second "Run Script" build phase.
  {{< image src="xcode-runscript.jpg" alt="Xcode Run Script" align="center" >}}

```cpp
"$OF_PATH"/addons/ofxTensorFlow2/scripts/macos_install_libs.sh "$TARGET_BUILD_DIR/$PRODUCT_NAME.app";
```

### YOLOv4

We can run a similar object recognition example in TensorFlow.

{{< alert context="info" icon="✌️" >}}
**Model Formats**

Note that the YOLO model that ships with ofxTensorFlow2 is an older version (v4), but we will use it anyway as it will be accurate enough for our purposes.

Different ML platforms use different model formats. TensorFlow has its own formats (usually with extension `.pb`) and OpenCV uses the `.onnx` format. While tools exist to convert models between formats, it is not worth the hassle for the purposes of this exercise.
{{< /alert >}}

Make sure to copy the `data/model` folder from the `ofxTensorFlow2/example_yolo_v4` example into the new app's data folder. We will also bring in the `ofxYolo.h` file into our new project and use it as-is to interpret the prediction results.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"
#include "ofxTensorFlow2.h"
#include "ofxYolo.h"

class ofApp : public ofBaseApp 
{
public:
  void setup();
  void update();
  void draw();

  ofVideoGrabber grabber;
  ofxYolo yolo;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  // Start video grabber.
  grabber.setup(640, 480);

  // Load the YOLOv4 model and the classes file.
  yolo.setup("model", "classes.txt");
}

void ofApp::update()
{
  grabber.update();
  if (grabber.isFrameNew())
  {
    // Get the grabber pixels reference.
    ofPixels& grabberPix = grabber.getPixels();
    // Feed the new frame into YOLO.
    yolo.setInput(grabberPix);
    // Run the classifier model.
    yolo.update();
  }
}

void ofApp::draw()
{
  ofSetColor(255);
  grabber.draw(0, 0);

  // Draw the detected objects on top of the video.
  ofNoFill();
  for (int i = 0; i < yolo.getObjects().size(); i++)
  {
    ofxYolo::Object& object = yolo.getObjects()[i];

    if (object.ident.index == 0)
    {
      // Person!
      ofSetColor(ofColor::blue);
    }
    else if (object.ident.index == 16) 
    { 
      // Dog!
      ofSetColor(ofColor::green);
    }
    else 
    { 
      // Anything else...
      ofSetColor(ofColor::red);
    }
    ofDrawRectangle(object.bbox);
    ofDrawBitmapStringHighlight(object.ident.text + " // " + ofToString(object.confidence, 2), object.bbox.getTopLeft());
  }
}
```

{{< image src="tf-yolo.png" alt="TensorFlow YOLOv4" width="600px" >}}
