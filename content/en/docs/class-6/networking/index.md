---
title: "Networking"
description: ""
lead: ""
date: 2022-10-27T19:17:06-04:00
lastmod: 2022-10-27T19:17:06-04:00
draft: false
images: []
menu:
  docs:
    parent: "class-6"
    identifier: "networking"
weight: 720
toc: true
---

openFrameworks applications often need to communicate information with other programs. These can be on the same machine or on a different one, they can be written in OF or using a different framework or language. These programs need to use a common protocol for communicating, to ensure that the messages reach their destination, and that the receiver knows how to decode them.

Using the network is a good choice for sending and receiving information, as most machines and frameworks already have the infrastructure to do so. We just need to add a layer on top of this infrastructure to come up with a communication system.

## TCP and UDP

TCP and UDP are the two most common protocols used for communicating over the network. You have probably heard of TCP or TCP/IP, which is what the components used to connect to the internet are called. TCP and UDP are very different, and each is better suited for a different use case.

The [Network](https://openframeworks.cc/ofBook/chapters/network.html) chapter in the ofBook offers a great explanation of the differences between the two.

A message destination is defined by an *IP address* and a *port*.

* The IP address is used to determine which machine to target.
* The port is used to determine which component (or app) on that machine the message should go to.
* Apps can communicate on the same machine using the special addresses `localhost` or `127.0.0.1`. Either of those mean "the current machine".

Messages are sent as *packets*.

* A packet is a small bit of data.
* Depending on the network and protocol, different packet sizes are supported.
* Messages are broken down into packets before going over the network, and reassembled on the receiving end.
* Each packet includes extra information like the size, order, source, destination, etc. in its *header*.

### Connection

* TCP is connection-based.
  * A connection must be established between the server and client before sending and receiving messages.
  * The server needs to start first, and the client connects to it.
  * This connection is sometimes called a handshake, where both server and client acknowledge each other's existence.
* UDP is connection-less.
  * Messages are sent directly from the sender to receiver.
  * There is no check that the receiver is ready or even exists before the message is sent.
  * Messages can be sent to many clients at once, using [broadcast or multicast](https://erg.abdn.ac.uk/users/gorry/course/intro-pages/uni-b-mcast.html) IP addresses.

### Communication

* TCP is stream-oriented.
  * Messages are split up into packets, which might be of different sizes, which come in continuously as a stream to the client.
  * Usually, a delimiter is added at the end of the message to inform the client where to split the stream to form messages. This delimiter is usually a null char `\0`.
* UDP is message-oriented.
  * Messages are sent as a single packet or set of packets, and have explicit boundaries.
  * A delimiter is not necessary, and a message is always received as a whole.

### Reliability

* TCP messages are guaranteed to be received.
  * The connection ensures that the client is available and ready.
  * The receiver has to acknowledge when they get a message. If the sender does not get this acknowledgment, it re-sends the message.
  * Packets are ordered, so they are received in the order they were sent.
* UDP messages can get lost.
  * There is no acknowledgement that a message has been received.
  * Messages can be received in any order.
  * Messages may never make it to their destination (this is also the case for TCP, but it is handled by resending the lost packets).

### Speed

* TCP communication is slower.
  * Connections, acknowledgements, and error-checking make the process of receiving a message slower.
* UDP communication is fast!
  * Minimal feedback keep this lean and low-latency.

<figure style="width:600px;height:1117px;display:block;margin:0 auto;">
<video src="lululemon-portrait.mov" controls width="100%"></video>
<figcaption><i>Lululemon Community Wall / Array of Stars.</i></figcaption>
</figure>

<figure style="width:600px;height:400px;display:block;margin:0 auto;">
<video src="humanhattan.mp4" controls width="100%"></video>
<figcaption><i>Humanhattan / BIG.</i></figcaption>
</figure>

## Networking in OF

### ofxNetwork

OF comes with the [ofxNetwork](https://openframeworks.cc/documentation/ofxNetwork/) addon.

* Includes everything you need for both TCP and UDP communication.
* Has great examples in the `path/to/OF/examples/communication/` directory to get started.

### ofxOsc

[Open Sound Control](http://opensoundcontrol.org/), or OSC, is a messaging protocol that was originally designed for communication between audio applications but has quickly become a standard for creative applications in general. OSC is based on UDP, meaning it is very fast and relies on messages for communication.

OSC is very useful because it tends to be supported out of the box in many applications and frameworks. Since it is network-based, it can be used to send messages between apps on a single computer or many different computers.

| Framework | Library |
|:----------|:--------|
| [Processing](https://processing.org/) | [oscP5](http://www.sojamo.de/libraries/oscp5/) |
| [p5.js](https://p5js.org/) | [p5js-osc](https://github.com/genekogan/p5js-osc) |
| [Unity](https://unity.com/) | [OSC simpl](https://assetstore.unity.com/packages/tools/input-management/osc-simpl-53710) |
| [Touch Designer](https://derivative.ca/) | [Built-in CHOP](https://docs.derivative.ca/OSC_In_CHOP) |
| [Max](https://cycling74.com/) | [OSC-route](http://cnmat.berkeley.edu/patch/4029) |

The OSC message format is also simple enough that it can be implemented manually. OSC messages have the following format:

```python
/user/defined/address arg0 arg1 arg2...
```

* Each component is separated by a space.
* The first component is the address, which is a series of words separated by `/`.
* The remaining components are the arguments:
  * There can be as many arguments as needed.
  * The arguments are simple data types, like `int`, `float`, `string`, etc.

In openFrameworks, we can use the built-in ofxOsc library to send and receive OSC messages.

Here is a simple sender that sends the mouse position when dragged, and a message with no arguments when the spacebar is pressed.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxOsc.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void draw();

  void keyPressed(int key);
  void mouseDragged(int x, int y, int button);

  std::string sendAddr;
  int sendPort;

  ofxOscSender sender;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  // Make the circle smoother by increasing the resolution.
  ofSetCircleResolution(128);

  // Set up the OSC sender.
  sendAddr = "localhost";
  sendPort = 3030;
  sender.setup(sendAddr, sendPort);
}

void ofApp::draw()
{
  ofBackground(0);

  // Draw a circle at the mouse position.
  if (ofGetMousePressed())
  {
      ofFill();
  }
  else
  {
      ofNoFill();
  }
  ofSetColor(255);
  ofDrawCircle(ofGetMouseX(), ofGetMouseY(), 50);
  ofFill();
}

void ofApp::keyPressed(int key)
{
  if (key == ' ')
  {
    ofxOscMessage msg;
    msg.setAddress("/cursor/color");

    sender.sendMessage(msg);
  }
}

void ofApp::mouseDragged(int x, int y, int button)
{
  ofxOscMessage msg;
  msg.setAddress("/cursor/move");
  msg.addIntArg(ofGetMouseX());
  msg.addIntArg(ofGetMouseY());

  sender.sendMessage(msg);
}
```

And here is the receiver counterpart, that positions a cursor at the received mouse position, and changes its color when the spacebar is pressed.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxOsc.h"

class ofApp : public ofBaseApp 
{

public:
  void setup();
  void update();
  void draw();

  int recvPort;

  ofxOscReceiver receiver;

  int cursorX;
  int cursorY;
  ofColor cursorColor;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofSetWindowShape(640, 480);

  // Make the circle smoother by increasing the resolution.
  ofSetCircleResolution(128);

  // Set up the OSC receiver.
  recvPort = 3030;
  receiver.setup(recvPort);
}

void ofApp::update()
{
  while (receiver.hasWaitingMessages())
  {
    // Get the next message.
    ofxOscMessage msg;
    receiver.getNextMessage(msg);

    if (msg.getAddress() == "/cursor/move")
    {
      cursorX = msg.getArgAsInt(0);
      cursorY = msg.getArgAsInt(1);
    }
    else if (msg.getAddress() == "/cursor/color")
    {
      // Generate a new random color.
      cursorColor = ofColor(ofRandom(127, 255), ofRandom(127, 255), ofRandom(127, 255));
    }
    else
    {
      ofLogWarning(__FUNCTION__) << "Unrecognized message " << msg.getAddress();
    }
  }
}

void ofApp::draw()
{
  ofBackground(0);

  // Draw a circle at the cursor position.
  ofSetColor(cursorColor);
  ofDrawCircle(cursorX, cursorY, 50);
}
```

Similar techniques can be used to send contour finder or face tracker data over to other applications for further processing.

<figure style="width:600px;height:400px;display:block;margin:0 auto;">
<video src="supervision.mp4" controls width="100%"></video>
<figcaption><i>Supervision / AMNH.</i></figcaption>
</figure>
<figure style="width:600px;height:400px;display:block;margin:0 auto;">
<video src="trex-alive.mp4" controls width="100%"></video>
<figcaption><i>T.rex Alive / AMNH.</i></figcaption>
</figure>

### Community Addons

* [ofxHTTP](https://github.com/bakercp/ofxHTTP) is a comprehensive community addon which is great for communicating with web servers.
  * Focused on HTTP, meaning it's TCP.
  * Can serve and connect to both HTTP and HTTPS servers.
* [ofxLibwebsockets](https://github.com/robotconscience/ofxLibwebsockets) is an OF wrapper for [WebSockets](http://www.websocket.org/).
  * Good documentation and examples.
  * Reliable protocol and implementation.
* [ofxZmq](https://github.com/jvcleave/ofxZmq) is an OF implementation of [ZeroMQ](https://zeromq.org/).
  * Optimized messaging that can be considered a reliable alternative to OSC.
* [Many, many more...](http://ofxaddons.com/categories/6-web-networking)
