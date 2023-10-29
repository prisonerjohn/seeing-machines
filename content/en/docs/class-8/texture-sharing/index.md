---
title: "Texture Sharing"
description: ""
lead: ""
date: 2022-11-04T15:46:35-04:00
lastmod: 2022-11-04T15:46:35-04:00
draft: false
images: []
menu:
  docs:
    parent: "class-7"
    identifier: "texture-sharing"
weight: 820
toc: true
---

It is sometimes necessary to share more than just state updates or short messages between applications. We may need to share an entire image over for various reasons:

* Continue image processing in another app.
* Use OF image as a texture for a material in a 3D rendering application.
* Projection map the OF image using mapping software.
* Use OF as an input for VJ software.
* Etc.

Texture sharing is often required for building interactive installations, and there are a few options for doing so. The following frameworks use special optimized techniques to share images, and therefore tend to be much faster than something built for shorter messages, like OSC for example.

<figure style="width:600px;height:400px;display:block;margin:0 auto;">
  <iframe width="600" height="375" src="https://www.youtube.com/embed/3UzpnA2nNlc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
  <figcaption><i>morookamitsuo Feb 14, 2015</i></figcaption>
</figure>

## Syphon

[Syphon](https://syphon.github.io/) is a macOS technology that allows applications to share frames in real-time. It does this by sharing texture memory avoiding unnecessary copies, and keeping data on the GPU as much as possible.

Syphon has plug-ins available for a multitude of platforms including [openFrameworks](https://github.com/astellato/ofxSyphon), [Processing](https://github.com/Syphon/Processing), [Unity](https://github.com/keijiro/KlakSyphon), [Jitter](https://github.com/Syphon/Jitter/releases/), [VDMX](https://vidvox.net/), [MadMapper](https://madmapper.com/), etc. It is an open-source project, which makes it easy for anyone to build their own bindings to Syphon, and this is why the list of supported applications keeps growing.

We will use the [ofxSyphon](https://github.com/astellato/ofxSyphon) addon for openFrameworks. This includes both server and client implementations, meaning we can send and receive images to / from any other application that supports Syphon.

### Server

The following is a simple thresholding app that includes two servers:

* `serverGrabber` publishes the input video using the `ofxSyphonServer.publishTexture()` function.
  * This takes a pointer to an `ofTexture` as an argument, so we will use the `&` operator before the variable to convert it to a pointer.
* `serverThreshold` publishes the result threshold image using `ofxSyphonServer.publishScreen()`.
  * `publishScreen()` sends whatever has been drawn on screen up to that point, so we need to draw `thresholdImg` before calling it.
  * Note that the GUI panel does not get sent to Syphon because it is drawn after the call to `publishScreen()`.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"
#include "ofxSyphon.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();
  
  ofVideoGrabber grabber;
  ofImage thresholdImg;
  
  ofxSyphonServer serverGrabber;
  ofxSyphonServer serverThreshold;
  
  ofParameter<int> thresholdVal;
  ofParameter<bool> sendGrabber;
  ofParameter<bool> sendThreshold;
  
  ofxPanel guiPanel;
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
  
  // Allocate threshold image (same size as video, single channel).
  thresholdImg.allocate(640, 480, OF_IMAGE_GRAYSCALE);
  
  // Set up Syphon servers.
  serverGrabber.setName("Video Input");
  serverThreshold.setName("Threshold Image");
  
  // Set parameters and GUI.
  thresholdVal.set("Threshold Val", 127, 0, 255);
  sendGrabber.set("Send Grabber", true);
  sendThreshold.set("Send Threshold", true);
  
  guiPanel.setup("Syphon Send", "settings.json");
  guiPanel.add(thresholdVal);
  guiPanel.add(sendGrabber);
  guiPanel.add(sendThreshold);
}

void ofApp::update()
{
  grabber.update();
  
  if (grabber.isFrameNew())
  {
    // Threshold video image.
    // Use references (&) when getting the ofPixels objects to
    // avoid unnecessary copies.
    ofPixels& videoPix = grabber.getPixels();
    ofPixels& thresholdPix = thresholdImg.getPixels();
    for (int row = 0; row < videoPix.getHeight(); row++)
    {
      for (int col = 0; col < videoPix.getWidth(); col++)
      {
        int pixVal = videoPix.getColor(col, row).getBrightness();
        if (pixVal < thresholdVal)
        {
          thresholdPix.setColor(col, row, ofColor(0));
        }
        else
        {
          thresholdPix.setColor(col, row, ofColor(255));
        }
      }
    }
      
    thresholdImg.update();
    
    if (sendGrabber)
    {
      // Send grabber texture.
      // This has to be a pointer, so we add & before the argument.
      serverGrabber.publishTexture(&grabber.getTexture());
    }
  }
}

void ofApp::draw()
{
  thresholdImg.draw(0, 0);
  
  if (sendThreshold)
  {
    // Send current screen, which is just the threshold image.
    serverThreshold.publishScreen();
  }
  
  guiPanel.draw();
}
```

### Client

We can now write a client app that reads the published textures.

* Note that each `ofxSyphonClient` needs to be set with a corresponding name to the `ofxSyphonServer`.
* We first need to call `ofxSyphonClient.setup()` then `ofxSyphonClient.set()` to set the server and app name. The server name is whatever we used in our sender ("Video Input" or "Threshold Image" above), and the app name is the name of the executable. In our case it would be something like "syphon-sendDebug".

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"
#include "ofxSyphon.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void draw();
  
  ofxSyphonClient clientGrabber;
  ofxSyphonClient clientThreshold;
  
  ofParameter<bool> recvGrabber;
  ofParameter<bool> recvThreshold;
  
  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);
  
  // Set up Syphon clients.
  clientGrabber.setup();
  clientGrabber.set("Video Input", "syphon-sendDebug");
  clientThreshold.setup();
  clientThreshold.set("Threshold Image", "syphon-sendDebug");
  
  // Set parameters and GUI.
  recvGrabber.set("Recv Grabber", true);
  recvThreshold.set("Recv Threshold", false);
  
  guiPanel.setup("Syphon Recv", "settings.json");
  guiPanel.add(recvGrabber);
  guiPanel.add(recvThreshold);
}

void ofApp::draw()
{
  if (recvGrabber)
  {
    // Draw grabber.
    clientGrabber.draw(0, 0);
  }
  else if (recvThreshold)
  {
    // Draw threshold.
    clientThreshold.draw(0, 0);
  }
  
  guiPanel.draw();
}
```

### Server Directory

While this client works, it is not ideal that we have to know the server name ahead of time because this is prone to errors and the name might change at any time. `ofxSyphon` also comes with an `ofxSyphonServerDirectory` which lists all available servers on the machine.

Let's use this to write a more robust client application. We will use an event listener to automatically call a function when the value of an `ofParameter` is changed, and set the client using this info.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"
#include "ofxSyphon.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void draw();
  
  void serverIndexChanged(int& val);
  
  ofxSyphonServerDirectory serverDirectory;
  ofxSyphonClient client;
  
  ofParameter<int> serverIdx;
  
  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);
  
  // Set up Syphon server directory.
  serverDirectory.setup();
  
  // Set up Syphon client.
  client.setup();
  
  // Set parameters and GUI.
  serverIdx.set("Server Idx", 0, 0, 10);
  serverIdx.addListener(this, &ofApp::serverIndexChanged);
  
  guiPanel.setup("Syphon Recv", "settings.json");
  guiPanel.add(serverIdx);
}

void ofApp::draw()
{
  if (serverDirectory.isValidIndex(serverIdx))
  {
    // Draw Syphon server.
    client.draw(0, 0);
  }
  
  guiPanel.draw();
}

void ofApp::serverIndexChanged(int& val)
{
  // Check that the server index is within bounds.
  if (serverDirectory.isValidIndex(serverIdx))
  {
    ofLogNotice(__FUNCTION__) << "Bind server " << serverIdx;
    
    // Bind the client to the selected server.
    client.set(serverDirectory.getDescription(serverIdx));
  }
  else
  {
    ofLogWarning(__FUNCTION__) << "Server " << serverIdx << " out of bounds!";
  }
}
```

### Event Listeners

Event listeners in openFrameworks are functions that are automatically called when an event happens. We have used pre-configured listeners like `mousePressed()` before, but we can also set up our own.

An event listener function first needs to be defined. This is simply a function that takes a reference to a variable as an argument. When using listeners to respond to changes to `ofParameter`, the argument needs to be the same as the `ofParameter` type.

For example, to listen for changes to an `ofParameter<int>`, our function argument would have to be a variable of type `int&`.

Once the function is defined, it needs to be registered with the object it is listening to.

* If we are dealing with an [`ofEvent`](https://openframeworks.cc///documentation/events/ofEvent/) directly, we can call [`ofEvent.add()`](https://openframeworks.cc/documentation/events/ofEvent/#show_add) or [`ofAddListener()`](https://openframeworks.cc/documentation/events/ofEventUtils/#!show_ofAddListener).
* If we are using an `ofParameter`, we call [`ofParameter.addListener()`](https://openframeworks.cc/documentation/types/ofParameter/#show_addListener).

In all cases, we need to pass a pointer to the object where the listener is defined, usually `this`, and a pointer to the listener function itself, usually something like `&ofApp:Listener`.

The following example adds listeners to the `serverAnnounced` and `serverRetired` events of `ofxSyphonServerDirectory` to ensure our index is always within range.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"
#include "ofxSyphon.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void draw();
  
  void serverListChanged(ofxSyphonServerDirectoryEventArgs& args);
  
  void serverIndexChanged(int& val);
  
  ofxSyphonServerDirectory serverDirectory;
  ofxSyphonClient client;
  
  ofParameter<int> serverIdx;
  
  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);
  
  // Set up Syphon server directory.
  serverDirectory.setup();
  ofAddListener(serverDirectory.events.serverAnnounced, this, &ofApp::serverListChanged);
  ofAddListener(serverDirectory.events.serverRetired, this, &ofApp::serverListChanged);

  // Set up Syphon client.
  client.setup();
  
  // Set parameters and GUI.
  serverIdx.set("Server Idx", 0, 0, 10);
  serverIdx.addListener(this, &ofApp::serverIndexChanged);
  
  guiPanel.setup("Syphon Recv", "settings.json");
  guiPanel.add(serverIdx);
  
  // Force call events once so that the app starts in a fully configured state.
  ofxSyphonServerDirectoryEventArgs args;
  serverListChanged(args);
  
  int idx = serverIdx;
  serverIndexChanged(idx);
}

void ofApp::draw()
{
  if (serverDirectory.isValidIndex(serverIdx))
  {
    // Draw Syphon server.
    client.draw(0, 0);
  }
  
  guiPanel.draw();
}

void ofApp::serverListChanged(ofxSyphonServerDirectoryEventArgs& args)
{
  // Adjust the range of the server index parameter.
  serverIdx.setMax(serverDirectory.size() - 1);
  
  // Make sure the server index is within bounds.
  if (serverIdx >= serverDirectory.size())
  {
    serverIdx = 0;
  }
}

void ofApp::serverIndexChanged(int& val)
{
  // Check that the server index is within bounds.
  if (serverDirectory.isValidIndex(serverIdx))
  {
    ofLogNotice(__FUNCTION__) << "Bind server " << serverIdx;
    
    ofxSyphonServerDescription desc = serverDirectory.getDescription(serverIdx);
    
    // Bind the client to the selected server.
    client.set(desc);
    
    // Update the window title.
    ofSetWindowTitle(desc.appName + "::" + desc.serverName);
  }
  else
  {
    ofLogWarning(__FUNCTION__) << "Server " << serverIdx << " out of bounds!";
  }
}
```

<figure style="width:600px;height:400px;display:block;margin:0 auto;" markdown="1">
<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/129234412?color=ef0065&title=0&byline=0&portrait=0" style="position:absolute;top:0;left:0;width:100%;height:100%;" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>
<figcaption><i><a href="https://vimeo.com/129234412">C4D_OF_ImageProcessing_SyphonLink</a> from <a href="https://vimeo.com/adamheslop">Adam Heslop</a> on <a href="https://vimeo.com">Vimeo</a>.</i></figcaption>
</figure>

## Spout

[Spout](http://spout.zeal.co/) is the equivalent to Syphon but for Windows platforms. It uses a special GPU feature that allows textures to be shared between different apps running on the same PC, and has a fallback method in case the hardware does not support it.

Similarly to Syphon, Spout already has plug-ins available for a multitude of platforms, and being open-source, allows any developer to write their own Spout interface for any tool they choose.

We will use the [ofxSpout](https://github.com/prisonerjohn/ofxSpout/tree/feature/upgrade-2.007h) addon for openFrameworks. Make sure to use the `upgrade-2.007h` branch of the addon.

### Sender

The following app is a similar thresholding app with two senders.

* Note that `ofxSpout` only includes a `send()` function which takes an `ofTexture` reference as a parameter.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"
#include "ofxSpout.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  ofVideoGrabber grabber;
  ofImage thresholdImg;

  ofxSpout::Sender senderGrabber;
  ofxSpout::Sender senderThreshold;

  ofParameter<int> thresholdVal;
  ofParameter<bool> sendGrabber;
  ofParameter<bool> sendThreshold;

  ofxPanel guiPanel;
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

  // Allocate threshold image (same size as video).
  thresholdImg.allocate(640, 480, OF_IMAGE_COLOR);

  // Set up Spout senders.
  senderGrabber.init("Video Input");
  senderThreshold.init("Threshold Image");

  // Set parameters and GUI.
  thresholdVal.set("Threshold Val", 127, 0, 255);
  sendGrabber.set("Send Grabber", true);
  sendThreshold.set("Send Threshold", true);

  guiPanel.setup("Spout Send", "settings.json");
  guiPanel.add(thresholdVal);
  guiPanel.add(sendGrabber);
  guiPanel.add(sendThreshold);
}

void ofApp::update()
{
  grabber.update();

  if (grabber.isFrameNew())
  {
    // Threshold video image.
    // Use references (&) when getting the ofPixels objects to
    // avoid unnecessary copies.
    ofPixels& videoPix = grabber.getPixels();
    ofPixels& thresholdPix = thresholdImg.getPixels();
    for (int row = 0; row < videoPix.getHeight(); row++)
    {
      for (int col = 0; col < videoPix.getWidth(); col++)
      {
        int pixVal = videoPix.getColor(col, row).getBrightness();
        if (pixVal < thresholdVal)
        {
          thresholdPix.setColor(col, row, ofColor(0));
        }
        else
        {
          thresholdPix.setColor(col, row, ofColor(255));
        }
      }
    }

    thresholdImg.update();

    if (sendGrabber)
    {
      // Send grabber texture.
      senderGrabber.send(grabber.getTexture());
    }

    if (sendThreshold)
    {
      // Send threshold texture.
      senderThreshold.send(thresholdImg.getTexture());
    }
  }
}

void ofApp::draw()
{
  thresholdImg.draw(0, 0);

  guiPanel.draw();
}
```

### Receiver

The following app holds two receivers corresponding to the two senders from above.

* We also need to create an `ofTexture` per receiver, and pass it as an argument to the `receive()` function.
* `ofxSpout::Receiver` holds a `selectSenderPanel()` function, which displays all available senders when called. Selecting a different sender in the panel overrides any settings made in the app.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"
#include "ofxSpout.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  void keyPressed(int key);

  ofxSpout::Receiver receiverGrabber;
  ofxSpout::Receiver receiverThreshold;

  ofTexture texGrabber;
  ofTexture texThreshold;

  ofParameter<bool> recvGrabber;
  ofParameter<bool> recvThreshold;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  // Set up Spout receivers.
  receiverGrabber.init("Video Input");
  receiverThreshold.init("Threshold Image");

  // Set parameters and GUI.
  recvGrabber.set("Recv Grabber", true);
  recvThreshold.set("Recv Threshold", false);

  guiPanel.setup("Spout Recv", "settings.json");
  guiPanel.add(recvGrabber);
  guiPanel.add(recvThreshold);
}

void ofApp::update()
{
  if (recvGrabber)
  {
    // Draw grabber.
    receiverGrabber.receive(texGrabber);
  }
  else if (recvThreshold)
  {
    // Draw threshold.
    receiverThreshold.receive(texThreshold);
  }
}

void ofApp::draw()
{
  if (recvGrabber)
  {
    // Draw grabber.
    texGrabber.draw(0, 0);
  }
  else if (recvThreshold)
  {
    // Draw threshold.
    texThreshold.draw(0, 0);
  }

  guiPanel.draw();
}

void ofApp::keyPressed(int key)
{
  if (key == ' ')
  {
    receiverGrabber.selectSenderPanel();
  }
}
```

## NDI

Syphon and Spout are great for sharing images between applications, but they are limited to a single machine. We sometimes need to share images between different machines and that's where NDI comes in!

[NDI](https://www.ndi.tv/) is a high performance standard for video over IP. This means that it can be used to send images over the network with very low latency. The SDK is compatible with Windows, Mac, Linux, and even iOS and Android making it truly versatile.

Depending on the addon we use, installing the NDI SDK is not necessary, but if we do, NDI will also advertise itself as a webcam. This means we could use an `ofVideoGrabber` as an NDI client directly in OF, we just need to select the device called "NewTek NDI Video".

While there are many options for using NDI under openFrameworks (see [here](https://github.com/thomasgeissl/ofxNDI), [here](https://github.com/leadedge/ofxNDI), and [here](https://github.com/nariakiiwatani/ofxNDI)), I have not been able to find a fully working addon for Mac and Windows.

[ofxNDI](https://github.com/leadedge/ofxNDI) by Lynn Jarvis seems to be the best choice at the moment, but has some limitations / extra steps:

* The NDI SDK needs to be installed on the machine (after registration on the [NDI website]((https://www.ndi.tv/))).
* The library needs to be manually added to each project on Mac.

### Sender

The following app only has a single `ofxNDIsender`, and switches between textures to send using the value of the `ofParameter` settings.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"
#include "ofxNDI.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  ofVideoGrabber grabber;
  ofImage thresholdImg;

  ofxNDIsender ndiSender;

  ofParameter<int> thresholdVal;
  ofParameter<bool> sendGrabber;
  ofParameter<bool> sendThreshold;

  ofxPanel guiPanel;
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

  // Allocate threshold image (same size as video, single channel).
  thresholdImg.allocate(640, 480, OF_IMAGE_COLOR);

  // Set up NDI senders.
  ndiSender.CreateSender("NDI Sender", grabber.getWidth(), grabber.getHeight());

  // Set parameters and GUI.
  thresholdVal.set("Threshold Val", 127, 0, 255);
  sendGrabber.set("Send Grabber", true);
  sendThreshold.set("Send Threshold", false);

  guiPanel.setup("NDI Send", "settings.json");
  guiPanel.add(thresholdVal);
  guiPanel.add(sendGrabber);
  guiPanel.add(sendThreshold);
}

void ofApp::update()
{
  grabber.update();

  if (grabber.isFrameNew())
  {
    // Threshold video image.
    // Use references (&) when getting the ofPixels objects to
    // avoid unnecessary copies.
    ofPixels& videoPix = grabber.getPixels();
    ofPixels& thresholdPix = thresholdImg.getPixels();
    for (int row = 0; row < videoPix.getHeight(); row++)
    {
      for (int col = 0; col < videoPix.getWidth(); col++)
      {
        int pixVal = videoPix.getColor(col, row).getBrightness();
        if (pixVal < thresholdVal)
        {
          thresholdPix.setColor(col, row, ofColor(0));
        }
        else
        {
          thresholdPix.setColor(col, row, ofColor(255));
        }
      }
    }

    thresholdImg.update();

    if (sendGrabber)
    {
      // Send grabber pixels.
      ndiSender.SendImage(grabber.getPixels());
    }
    else if (sendThreshold)
    {
      // Send threshold pixels.
      ndiSender.SendImage(thresholdPix);
    }
  }
}

void ofApp::draw()
{
  thresholdImg.draw(0, 0);

  guiPanel.draw();
}
```

### Receiver

Because we only have a single sender, the receiver is simplified as a single `ofxNDIreceiver` that just gets drawn to the screen.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"
#include "ofxNDI.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void update();
  void draw();

  ofxNDIreceiver ndiReceiver;
  ofTexture ndiTexture;

  ofxPanel guiPanel;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  ndiTexture.allocate(640, 480, GL_RGBA);

  // Set parameters and GUI.
  guiPanel.setup("NDI Recv", "settings.json");
}

void ofApp::update()
{
  ndiReceiver.ReceiveImage(ndiTexture);
}

void ofApp::draw()
{
  ndiTexture.draw(0, 0);

  guiPanel.draw();
}
```

If there is enough interest in the class to use NDI, I can try to update one of the addons to work across the board, just let me know!

<figure style="width:600px;height:400px;display:block;margin:0 auto;">
<video src="unity-vision.mp4" controls muted width="100%"></video>
<figcaption><i>Unity Vision.</i></figcaption>
</figure>
