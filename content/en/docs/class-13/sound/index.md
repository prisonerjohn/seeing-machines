---
title: "Sound"
description: ""
lead: ""
date: 2023-11-22T14:33:17-05:00
lastmod: 2023-11-22T14:33:17-05:00
draft: true
images: []
menu:
  docs:
    parent: "class-13"
    identifier: "sound"
weight: 1310
toc: true
---

Almost all the material we've covered so far has dealt with computer vision. The following notes deal with another important "sense", sound.

openFrameworks might not seem like the obvious choice for manipulating sound. Part of the issue is that development of the sound modules in OF have been stop-and-go since the beginning; especially compared to other tools like [Max](https://cycling74.com/) which place audio front and center, and have a more visual approach to working with sound data and devices. That being said, sound analysis and manipulation is still quite developed in OF and involves some interesting concepts that should be explored.

## Sound Stream

The primary object for working with sound is [`ofSoundStream`](https://openframeworks.cc/documentation/sound/ofSoundStream/). An `ofSoundStream` controls access to the computer's audio input and output devices, and allows the manipulation of the sound data in real-time.

Typical apps will only use a single sound stream, and openFrameworks gives you easy access to a default `ofSoundStream` using `ofSoundStreamXXX()` global functions like [`ofSoundStreamSetup()`](https://openframeworks.cc/documentation/sound/ofSoundStream/#show_ofSoundStreamSetup) or [`ofSoundStreamClose()`](https://openframeworks.cc/documentation/sound/ofSoundStream/#show_ofSoundStreamClose). However, we like to be explicit in this class so our examples will always create our own `ofSoundStream` object :)

An `ofSoundStream` is set up with an [`ofSoundStreamSettings`](https://openframeworks.cc/documentation/sound/ofSoundStreamSettings/). This sets the sound hardware to use, the buffer properties like number of channels and sample rate, the callbacks for reading and writing sound data, etc.

```cpp
ofSoundStreamSettings settings;
settings.sampleRate = 44100;
settings.bufferSize = 256;
settings.numOutputChannels = 2;
settings.numInputChannels = 0;

soundStream.setup(settings);
```

We can list the installed sound devices on our system using [`ofSoundStream::printDeviceList()`](https://openframeworks.cc/documentation/sound/ofSoundStream/#show_printDeviceList). This will print all the information out to the console, including the interface used and the number of available input and output channels per device.

The output will look something like:

```python
[notice ] ofBaseSoundStream::printDeviceList: Api: MS WASAPI
[notice ] ofBaseSoundStream::printDeviceList: [MS WASAPI: 0] DELL U2713HM (NVIDIA High Definition Audio) [in:0 out:2]
[MS WASAPI: 1] Speakers (Realtek High Definition Audio) [in:0 out:2] (default out)
[MS WASAPI: 2] Realtek Digital Output(Optical) (Realtek High Definition Audio) [in:0 out:2]
[MS WASAPI: 3] Realtek Digital Output (Realtek High Definition Audio) [in:0 out:2]
[MS WASAPI: 4] Microphone (HD Pro Webcam C920) [in:2 out:0]
[MS WASAPI: 5] Microphone Array (Xbox NUI Sensor) [in:4 out:0] (default in)
```

It is a good idea to close the sound stream with [`ofSoundStream::close()`](https://openframeworks.cc/documentation/sound/ofSoundStream/#show_close) when we are finished with it as that will prevent our app from receiving unwanted sound bytes. We will do this in `ofApp::exit()` in the following examples.

## Sound Buffer

Sound data is packed into an [`ofSoundBuffer`](https://openframeworks.cc/documentation/sound/ofSoundBuffer/).

As the name implies, `ofSoundBuffer` includes the sound buffer data.

* Buffer values are `float`, with range from `-1` to `1`. Think of this as a waveform, with `0` at the origin.
* This can be used like an array, using the [`[]`](https://openframeworks.cc/documentation/sound/ofSoundBuffer/#show_operator%5B%5D) operator and an index to access each element.
* Just like an `ofPixels` array, the sound data is *interleaved* in the buffer. This means that if we have 2 channels of audio, it will be packed as `LRLRLRLRLR...`.
* [`size()`](https://openframeworks.cc/documentation/sound/ofSoundBuffer/#show_size) returns the total buffer size.
* [`getNumChannels()`](https://openframeworks.cc/documentation/sound/ofSoundBuffer/#show_getNumChannels) returns the number of channels.
* [`getNumFrames()`](https://openframeworks.cc/documentation/sound/ofSoundBuffer/#show_getNumFrames) returns the number of frames, which is the buffer size per channel. This is essentially the size divided by the number of channels.

`ofSoundBuffer` also includes a few other getters and setters:

* Some to interface with the audio hardware, like [`getDeviceID()`](https://openframeworks.cc/documentation/sound/ofSoundBuffer/#show_getDeviceID) and [`setSampleRate()`](https://openframeworks.cc/documentation/sound/ofSoundBuffer/#show_setSampleRate).
* Some to manipulate the signal, like [`fillWithNoise()`](https://openframeworks.cc/documentation/sound/ofSoundBuffer/#show_fillWithNoise) and [`resample()`](https://openframeworks.cc/documentation/sound/ofSoundBuffer/#show_resample).

## Sound Input

To receive audio in OF, we need to:

* Set the input channels of our `ofSoundStream` to a value greater than `0`.
* Set an audio listener to the `ofSoundStream` using [`ofSoundStreamSettings::setInListener()`](https://openframeworks.cc/documentation/sound/ofSoundStreamSettings/#show_setInListener). This can be any class that extends from `ofBaseSoundInput`, which includes our `ofApp`.
* Implement the [`audioIn()`](https://openframeworks.cc/documentation/types/ofBaseSoundInput/#show_audioIn) function in our listener class.

If the app does not receive any audio and prints out messages like `RtAudio: no compiled support for specified API argument!`, you may not be using the right sound device.

* Review the printed output from the `ofSoundStream::printDeviceList()` call and select an appropriate device.
* Set the corresponding API with `ofSoundStreamSettings::setApi()`.
* Set the corresponding device ID with `ofSoundStreamSettings::setInDevice()` or directly in the sound stream with `ofSoundStream::setDeviceID()`.

```cpp
//ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void exit();
  void update();
  void draw();

  void audioIn(ofSoundBuffer& input);

  ofSoundStream soundStream;

  ofxPanel guiPanel;
};
```

```cpp
//ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  // Print audio device list to the console.
  soundStream.printDeviceList();
  //soundStream.setDeviceID(1);

  // Setup sound stream.
  ofSoundStreamSettings settings;
  //settings.setApi(ofSoundDevice::Api::MS_DS);
  settings.setInListener(this);
  //settings.sampleRate = 44100;
  settings.numOutputChannels = 0;
  settings.numInputChannels = 2;
  soundStream.setup(settings);

  guiPanel.setup("Audio In", "settings.json");
}

void ofApp::exit()
{
  soundStream.close();
}

void ofApp::update() 
{

}

void ofApp::draw() 
{
  guiPanel.draw();
}

void ofApp::audioIn(ofSoundBuffer& input)
{

}
```

### RMS

To calculate audio levels, we need to consider all samples in the received buffer. We could try adding and averaging all values in our buffer, but since the values oscillate between `-1` and `1`, we would probably always end up with something near `0` (since the negatives will cancel out the positives). A good way to approximate the audio level is to calculate the root mean square (RMS), which ensures that the samples collected are always positive.

```cpp
//ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void exit();
  void update();
  void draw();

  void audioIn(ofSoundBuffer& input);

  ofSoundStream soundStream;

  ofParameter<float> smoothPct;
  ofParameter<float> volPeak;

  ofParameter<float> currVol;
  ofParameter<float> smoothVol;
  ofParameter<float> scaledVol;

  ofxPanel guiPanel;
};
```

```cpp
//ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  // Print audio device list to the console.
  soundStream.printDeviceList();
  //soundStream.setDeviceID(3);

  // Setup sound stream.
  ofSoundStreamSettings settings;
  //settings.setApi(ofSoundDevice::Api::MS_DS);
  settings.setInListener(this);
  //settings.sampleRate = 44100;
  settings.numOutputChannels = 0;
  settings.numInputChannels = 2;
  soundStream.setup(settings);

  // Use ofParameter for volume values so that we can print them in the GUI.
  currVol.set("Curr Vol", 0.0, 0.0, 1.0);
  smoothVol.set("Smooth Vol", 0.0, 0.0, 1.0);
  scaledVol.set("Scaled Vol", 0.0, 0.0, 1.0);

  smoothPct.set("Smooth Pct", 0.05, 0.0, 1.0);
  volPeak.set("Vol Peak", 0.001, 0.0, 1.0);

  guiPanel.setup("Audio In", "settings.json");
  guiPanel.add(currVol);
  guiPanel.add(smoothVol);
  guiPanel.add(scaledVol);
  guiPanel.add(smoothPct);
  guiPanel.add(volPeak);
}

void ofApp::exit()
{
  soundStream.close();
}

void ofApp::update() 
{
  // Scale the volume up to a 0-1 range.
  volPeak = max(volPeak, smoothVol);
  scaledVol = ofMap(smoothVol, 0.0, volPeak, 0.0, 1.0, true);
}

void ofApp::draw() 
{
  float volWidth = ofMap(scaledVol, 0.0, 1.0, 0.0, ofGetWidth());
  ofSetColor(200, 0, 0);
  ofFill();
  ofDrawRectangle(0, ofGetHeight() - 120, volWidth, 120);
  ofSetColor(0);
  ofNoFill();
  ofDrawRectangle(0, ofGetHeight() - 120, ofGetWidth(), 120);

  guiPanel.draw();
}

void ofApp::audioIn(ofSoundBuffer& input)
{
  // Calculate the volume using RMS.
  currVol = 0.0;
  int numCounted = 0;
  for (int i = 0; i < input.getNumFrames(); i++)
  {
    // Samples should be between -1 and 1, but sometimes go a bit haywire 
    // when there's no data, so let's clamp the values to avoid outliers.
    float leftSample = ofClamp(input[i * 2 + 0], -1, 1) * 0.5;
    float rightSample = ofClamp(input[i * 2 + 1], -1, 1) * 0.5;

    currVol += leftSample * leftSample;
    currVol += rightSample * rightSample;
    
    numCounted += 2;
  }

  currVol /= (float)numCounted;
  currVol = sqrt(currVol);

  smoothVol = ofLerp(smoothVol, currVol, smoothPct);
}
```

In the above example, note that:

* The RMS value is very volatile, so we smooth it out using `ofLerp()`.
* The RMS value does not necessarily go up to `1.0` at it's peak, so we scale it up using `ofMap()` and its peak recorded value.
* Buffer samples sometimes go out of range. I'm not sure why but I think it occurs when no data is present, and only on specific hardware. This needs to be cleaned up before making any calculations, so we adjust outlier values using `ofClamp()`.

If we draw a graph of what our raw samples look like, we can indeed see that the tail end values seem to shoot up.

```cpp
//ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void exit();
  void update();
  void draw();

  void audioIn(ofSoundBuffer& input);

  ofSoundStream soundStream;

  std::vector<float> leftSamples;
  std::vector<float> rightSamples;

  ofParameter<float> smoothPct;
  ofParameter<float> volPeak;

  ofParameter<float> currVol;
  ofParameter<float> smoothVol;
  ofParameter<float> scaledVol;

  ofxPanel guiPanel;
};
```

```cpp
//ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofBackground(0);

  // Print audio device list to the console.
  soundStream.printDeviceList();
  //soundStream.setDeviceID(3);

  // Setup sound stream.
  ofSoundStreamSettings settings;
  //settings.setApi(ofSoundDevice::Api::MS_DS);
  settings.setInListener(this);
  //settings.sampleRate = 44100;
  settings.bufferSize = 512;
  settings.numOutputChannels = 0;
  settings.numInputChannels = 2;
  soundStream.setup(settings);

  // Initialize sample history.
  leftSamples.assign(settings.bufferSize, 0.0);
  rightSamples.assign(settings.bufferSize, 0.0);

  // Use ofParameter for volume values so that we can print them in the GUI.
  currVol.set("Curr Vol", 0.0, 0.0, 1.0);
  smoothVol.set("Smooth Vol", 0.0, 0.0, 1.0);
  scaledVol.set("Scaled Vol", 0.0, 0.0, 1.0);

  smoothPct.set("Smooth Pct", 0.05, 0.0, 1.0);
  volPeak.set("Vol Peak", 0.001, 0.0, 1.0);

  guiPanel.setup("Audio In", "settings.json");
  guiPanel.add(currVol);
  guiPanel.add(smoothVol);
  guiPanel.add(scaledVol);
  guiPanel.add(smoothPct);
  guiPanel.add(volPeak);
}

void ofApp::exit()
{
  soundStream.close();
}

void ofApp::update() 
{
  // Scale the volume up to a 0-1 range.
  volPeak = max(volPeak, smoothVol);
  scaledVol = ofMap(smoothVol, 0.0, volPeak, 0.0, 1.0, true);
}

void ofApp::draw() 
{
  ofSetLineWidth(2.0);

  // Draw the left channel history.
  ofSetColor(200, 0, 0);
  ofBeginShape();
  for (int i = 0; i < leftSamples.size(); i++) 
  {
    float x = ofMap(i, 0, leftSamples.size(), 0, ofGetWidth());
    float y = ofMap(leftSamples[i], -1.0, 1.0, 0, 200);
    ofVertex(x, y);
  }
  ofEndShape(false);
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, 0, ofGetWidth(), 200);

  // Draw the right channel history.
  ofSetColor(200, 0, 0);
  ofBeginShape();
  for (int i = 0; i < rightSamples.size(); i++)
  {
    float x = ofMap(i, 0, rightSamples.size(), 0, ofGetWidth());
    float y = ofMap(rightSamples[i], -1.0, 1.0, 200, 400);
    ofVertex(x, y);
  }
  ofEndShape(false);
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, 200, ofGetWidth(), 200);

  // Draw the volume level.
  float volWidth = ofMap(scaledVol, 0.0, 1.0, 0.0, ofGetWidth());
  ofSetColor(200, 0, 0);
  ofFill();
  ofDrawRectangle(0, ofGetHeight() - 120, volWidth, 120);
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, ofGetHeight() - 120, ofGetWidth(), 120);

  guiPanel.draw();
}

void ofApp::audioIn(ofSoundBuffer& input)
{
  // Calculate the volume using RMS.
  currVol = 0.0;
  int numCounted = 0;
  for (int i = 0; i < input.getNumFrames(); i++)
  {
    // Samples should be between -1 and 1, but sometimes go a bit haywire 
    // when there's no data, so let's clamp the values to avoid outliers.
    leftSamples[i] = ofClamp(input[i * 2 + 0], -1, 1) * 0.5;
    rightSamples[i] = ofClamp(input[i * 2 + 1], -1, 1) * 0.5;

    currVol += leftSamples[i] * leftSamples[i];
    currVol += rightSamples[i] * rightSamples[i];
    
    numCounted += 2;
  }

  currVol /= (float)numCounted;
  currVol = sqrt(currVol);

  smoothVol = ofLerp(smoothVol, currVol, smoothPct);
}
```

{{< image src="sound-graph.png" alt="Sound Graph" width="600px" >}}

### Onset Detection

We can use the RMS for onset or beat detection. We can set a threshold value and every time the RMS goes above it, a new beat is recorded. We can then use this to have an action happen to the tempo of the audio.

```cpp
//ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void exit();
  void update();
  void draw();

  void audioIn(ofSoundBuffer& input);

  ofSoundStream soundStream;

  std::vector<float> leftSamples;
  std::vector<float> rightSamples;

  ofParameter<float> smoothPct;
  ofParameter<float> volPeak;

  ofParameter<float> currVol;
  ofParameter<float> smoothVol;
  ofParameter<float> scaledVol;

  ofParameter<float> onsetThreshold;

  ofxPanel guiPanel;
};
```

```cpp
//ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofBackground(0);

  // Print audio device list to the console.
  soundStream.printDeviceList();
  //soundStream.setDeviceID(3);

  // Setup sound stream.
  ofSoundStreamSettings settings;
  //settings.setApi(ofSoundDevice::Api::MS_DS);
  settings.setInListener(this);
  //settings.sampleRate = 44100;
  settings.bufferSize = 512;
  settings.numOutputChannels = 0;
  settings.numInputChannels = 2;
  soundStream.setup(settings);

  // Initialize sample history.
  leftSamples.assign(settings.bufferSize, 0.0);
  rightSamples.assign(settings.bufferSize, 0.0);

  // Use ofParameter for volume values so that we can print them in the GUI.
  currVol.set("Curr Vol", 0.0, 0.0, 1.0);
  smoothVol.set("Smooth Vol", 0.0, 0.0, 1.0);
  scaledVol.set("Scaled Vol", 0.0, 0.0, 1.0);

  smoothPct.set("Smooth Pct", 0.05, 0.0, 1.0);
  volPeak.set("Vol Peak", 0.001, 0.0, 1.0);

  onsetThreshold.set("Onset Thresh", 0.5, 0.0, 1.0);

  guiPanel.setup("Audio In", "settings.json");
  guiPanel.add(currVol);
  guiPanel.add(smoothVol);
  guiPanel.add(scaledVol);
  guiPanel.add(smoothPct);
  guiPanel.add(volPeak);
  guiPanel.add(onsetThreshold);
}

void ofApp::exit()
{
  soundStream.close();
}

void ofApp::update() 
{
  // Scale the volume up to a 0-1 range.
  volPeak = max(volPeak, smoothVol);
  scaledVol = ofMap(smoothVol, 0.0, volPeak, 0.0, 1.0, true);
}

void ofApp::draw() 
{
  ofSetLineWidth(2.0);

  // Draw the left channel history.
  ofSetColor(200, 0, 0);
  ofBeginShape();
  for (int i = 0; i < leftSamples.size(); i++) 
  {
    float x = ofMap(i, 0, leftSamples.size(), 0, ofGetWidth());
    float y = ofMap(leftSamples[i], -1.0, 1.0, 0, 200);
    ofVertex(x, y);
  }
  ofEndShape(false);
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, 0, ofGetWidth(), 200);

  // Draw the right channel history.
  ofSetColor(200, 0, 0);
  ofBeginShape();
  for (int i = 0; i < rightSamples.size(); i++)
  {
    float x = ofMap(i, 0, rightSamples.size(), 0, ofGetWidth());
    float y = ofMap(rightSamples[i], -1.0, 1.0, 200, 400);
    ofVertex(x, y);
  }
  ofEndShape(false);
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, 200, ofGetWidth(), 200);

  // Draw the volume level.
  float volWidth = ofMap(scaledVol, 0.0, 1.0, 0.0, ofGetWidth());
  ofFill();
  if (scaledVol < onsetThreshold)
  {
    ofSetColor(200, 0, 0);
  }
  else
  {
    ofSetColor(0, 200, 0);
  }
  ofDrawRectangle(0, ofGetHeight() - 120, volWidth, 120);
  // Draw the threshold level.
  ofSetColor(225);
  float threshPos = ofMap(onsetThreshold, 0.0, 1.0, 0.0, ofGetWidth());
  ofDrawRectangle(threshPos, ofGetHeight() - 120, 2, 120);
  // Draw a border around the box.
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, ofGetHeight() - 120, ofGetWidth(), 120);

  guiPanel.draw();
}

void ofApp::audioIn(ofSoundBuffer& input)
{
  // Calculate the volume using RMS.
  currVol = 0.0;
  int numCounted = 0;
  for (int i = 0; i < input.getNumFrames(); i++)
  {
    // Samples should be between -1 and 1, but sometimes go a bit haywire 
    // when there's no data, so let's clamp the values to avoid outliers.
    leftSamples[i] = ofClamp(input[i * 2 + 0], -1, 1) * 0.5;
    rightSamples[i] = ofClamp(input[i * 2 + 1], -1, 1) * 0.5;

    currVol += leftSamples[i] * leftSamples[i];
    currVol += rightSamples[i] * rightSamples[i];
    
    numCounted += 2;
  }

  currVol /= (float)numCounted;
  currVol = sqrt(currVol);

  smoothVol = ofLerp(smoothVol, currVol, smoothPct);
}
```

### FFT

Onset detection works fine when you have fairly isolated audio, like footsteps or a bass drum beat, but degenerates fairly quickly when the sound source becomes more complex and involves multiple frequencies.

The Fast Fourier Transform (FFT) is an algorithm used to take in a sound sample and compute the levels for an array of frequencies. This essentially splits up the audio into different bands that can be analyzed separately.

FFT does not work out of the box with OF sound input, so we will need to use the [ofxFft](https://github.com/kylemcdonald/ofxFft) addon. This addon includes a few examples to get going as well as two different FFT implementations, but we'll just use the default for this example.

```cpp
//ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxFft.h"
#include "ofxGui.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void exit();
  void update();
  void draw();

  void audioIn(ofSoundBuffer& input);

  ofSoundStream soundStream;

  ofxFft* fft;
  std::vector<float> amplitudeBands;

  std::vector<float> leftSamples;
  std::vector<float> rightSamples;

  ofParameter<float> smoothPct;
  ofParameter<float> volPeak;

  ofParameter<float> currVol;
  ofParameter<float> smoothVol;
  ofParameter<float> scaledVol;

  ofParameter<float> onsetThreshold;

  ofxPanel guiPanel;
};
```

```cpp
//ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofBackground(0);

  // Print audio device list to the console.
  soundStream.printDeviceList();
  //soundStream.setDeviceID(3);

  // Setup sound stream.
  ofSoundStreamSettings settings;
  //settings.setApi(ofSoundDevice::Api::MS_DS);
  settings.setInListener(this);
  //settings.sampleRate = 44100;
  settings.bufferSize = 64;
  settings.numOutputChannels = 0;
  settings.numInputChannels = 2;
  soundStream.setup(settings);

  // Setup FFT.
  fft = ofxFft::create(settings.bufferSize, OF_FFT_WINDOW_HAMMING);
  amplitudeBands.resize(settings.bufferSize, 0.0f);

  // Initialize sample history.
  leftSamples.assign(settings.bufferSize, 0.0);
  rightSamples.assign(settings.bufferSize, 0.0);

  // Use ofParameter for volume values so that we can print them in the GUI.
  currVol.set("Curr Vol", 0.0, 0.0, 1.0);
  smoothVol.set("Smooth Vol", 0.0, 0.0, 1.0);
  scaledVol.set("Scaled Vol", 0.0, 0.0, 1.0);

  smoothPct.set("Smooth Pct", 0.05, 0.0, 1.0);
  volPeak.set("Vol Peak", 0.001, 0.0, 1.0);

  onsetThreshold.set("Onset Thresh", 0.5, 0.0, 1.0);

  guiPanel.setup("Audio In", "settings.json");
  guiPanel.add(currVol);
  guiPanel.add(smoothVol);
  guiPanel.add(scaledVol);
  guiPanel.add(smoothPct);
  guiPanel.add(volPeak);
  guiPanel.add(onsetThreshold);
}

void ofApp::exit()
{
  soundStream.close();
}

void ofApp::update() 
{
  // Scale the volume up to a 0-1 range.
  volPeak = max(volPeak, smoothVol);
  scaledVol = ofMap(smoothVol, 0.0, volPeak, 0.0, 1.0, true);
}

void ofApp::draw() 
{
  ofSetLineWidth(2.0);

  // Draw the left channel history.
  ofSetColor(200, 0, 0);
  ofBeginShape();
  for (int i = 0; i < leftSamples.size(); i++) 
  {
    float x = ofMap(i, 0, leftSamples.size(), 0, ofGetWidth());
    float y = ofMap(leftSamples[i], -1.0, 1.0, 0, 200);
    ofVertex(x, y);
  }
  ofEndShape(false);
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, 0, ofGetWidth(), 200);

  // Draw the right channel history.
  ofSetColor(200, 0, 0);
  ofBeginShape();
  for (int i = 0; i < rightSamples.size(); i++)
  {
    float x = ofMap(i, 0, rightSamples.size(), 0, ofGetWidth());
    float y = ofMap(rightSamples[i], -1.0, 1.0, 200, 400);
    ofVertex(x, y);
  }
  ofEndShape(false);
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, 200, ofGetWidth(), 200);

  // Draw the frequency spectrum (FFT).
  ofSetColor(0, 0, 200);
  ofFill();
  float bandWidth = ofGetWidth() / fft->getBinSize();
  for (int i = 0; i < fft->getBinSize(); i++)
  {
    // Negative height to draw from the bottom up.
    ofDrawRectangle(i * bandWidth, 600, bandWidth, amplitudeBands[i] * -200);
  }
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, 400, ofGetWidth(), 200);

  // Draw the volume level.
  float volWidth = ofMap(scaledVol, 0.0, 1.0, 0.0, ofGetWidth());
  ofFill();
  if (scaledVol < onsetThreshold)
  {
    ofSetColor(200, 0, 0);
  }
  else
  {
    ofSetColor(0, 200, 0);
  }
  ofDrawRectangle(0, ofGetHeight() - 120, volWidth, 120);
  // Draw the threshold level.
  ofSetColor(225);
  float threshPos = ofMap(onsetThreshold, 0.0, 1.0, 0.0, ofGetWidth());
  ofDrawRectangle(threshPos, ofGetHeight() - 120, 2, 120);
  // Draw a border around the box.
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, ofGetHeight() - 120, ofGetWidth(), 120);

  guiPanel.draw();
}

void ofApp::audioIn(ofSoundBuffer& input)
{
  // Calculate the volume using RMS.
  currVol = 0.0;
  int numCounted = 0;
  for (int i = 0; i < input.getNumFrames(); i++)
  {
    // Samples should be between -1 and 1, but sometimes go a bit haywire 
    // when there's no data, so let's clamp the values to avoid outliers.
    leftSamples[i] = ofClamp(input[i * 2 + 0], -1, 1) * 0.5;
    rightSamples[i] = ofClamp(input[i * 2 + 1], -1, 1) * 0.5;

    currVol += leftSamples[i] * leftSamples[i];
    currVol += rightSamples[i] * rightSamples[i];
    
    numCounted += 2;
  }
  currVol /= (float)numCounted;
  currVol = sqrt(currVol);
  // Smooth volume over time.
  smoothVol = ofLerp(smoothVol, currVol, smoothPct);

  // Calculate amplitudes.
  fft->setSignal(input.getBuffer());
  float* amplitude = fft->getAmplitude();
  // Scale FFT bands to 0-1 range.
  float maxValue = 0;
  for (int i = 0; i < fft->getBinSize(); i++)
  {
    if (abs(amplitude[i]) > maxValue)
    {
      maxValue = abs(amplitude[i]);
    }
  }
  // Smooth FFT bands over time.
  for (int i = 0; i < fft->getBinSize(); i++)
  {
    if (maxValue > 0)
    {
      amplitude[i] /= maxValue;
    }
    amplitudeBands[i] = ofLerp(amplitudeBands[i], amplitude[i], smoothPct);
  }
}
```

{{< image src="sound-fft.png" alt="Sound FFT" width="600px" >}}

## Sound Player

[`ofSoundPlayer`](https://openframeworks.cc///documentation/sound/ofSoundPlayer/) is used to play sound files. The OF examples do a good job of showing all the functionality here, so I won't spend time on it here.

## Sound Synthesis

Generating sound in OF also starts with an `ofSoundStream`:

* Set the output channels of our `ofSoundStream` to a value greater than `0`.
* Set an audio listener to the `ofSoundStream` using [`ofSoundStreamSettings.setOutListener()`](https://openframeworks.cc/documentation/sound/ofSoundStreamSettings/#show_setOutListener). This can be any class that extends from `ofBaseSoundOutput`, which includes our `ofApp`.
* Implement the [`audioOut()`](https://openframeworks.cc/documentation/types/ofBaseSoundOutput/#!show_audioOut) function in our listener class.

```cpp
//ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"

class ofApp : public ofBaseApp 
{
public:
  void setup();
  void update();
  void draw();

  void audioOut(ofSoundBuffer& buffer);

  ofSoundStream soundStream;

  ofxPanel guiPanel;
};
```

```cpp
//ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofBackground(0);

  ofSoundStreamSettings settings;
  //settings.setApi(ofSoundDevice::Api::MS_DS);
  settings.setOutListener(this);
  //settings.sampleRate = 44100;
  settings.bufferSize = 256;
  settings.numOutputChannels = 2;
  settings.numInputChannels = 0;
  soundStream.setup(settings);

  guiPanel.setup("Audio Out", "settings.json");
}

void ofApp::update()
{

}

void ofApp::draw()
{
  guiPanel.draw();
}

void ofApp::audioOut(ofSoundBuffer& buffer)
{

}
```

### Signal Generator

We need to fill the buffer received in `audioOut()` with data to produce sound.

* We will generate a sine wave and send it to both left and right channels.
* The pan (balance between left and right channels) will be updated using the mouse x position.
* The phase (length of sine wave period) will be updated using the mouse y position. The shorter the phase, the higher the sound frequency.
* We want the sine wave to be continuous, so we'll use a class variable `phaseVal` to keep track of the value across frames.
* We want to avoid any jumps in phase values because these might lead to "pops" in the sound, so `phaseStep` will change gradually using `ofLerp()`.

```cpp
//ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"

class ofApp : public ofBaseApp 
{
public:
  void setup();
  void update();
  void draw();

  void audioOut(ofSoundBuffer& buffer);

  ofSoundStream soundStream;

  std::vector<float> leftSamples;
  std::vector<float> rightSamples;

  float phaseVal;
  float phaseStep;

  ofParameter<float> volume;
  ofParameter<float> pan;
  ofParameter<float> phaseSpeed;

  ofxPanel guiPanel;
};
```

```cpp
//ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofBackground(0);

  ofSoundStreamSettings settings;
  //settings.setApi(ofSoundDevice::Api::MS_DS);
  settings.setOutListener(this);
  //settings.sampleRate = 44100;
  settings.bufferSize = 256;
  settings.numOutputChannels = 2;
  settings.numInputChannels = 0;
  soundStream.setup(settings);

  phaseVal = 0;
  phaseStep = 0.0f;

  leftSamples.assign(settings.bufferSize, 0.0);
  rightSamples.assign(settings.bufferSize, 0.0);

  volume.set("Volume", 0.1, 0.0, 1.0);
  pan.set("Pan", 0.1, 0.0, 1.0);
  phaseSpeed.set("Phase Speed", 0.15, 0.0, 1.0);

  guiPanel.setup("Audio Out", "settings.json");
  guiPanel.add(volume);
  guiPanel.add(pan);
  guiPanel.add(phaseSpeed);
}

void ofApp::update()
{
  pan = ofMap(ofGetMouseX(), 0, ofGetWidth(), 0, 1, true);
  float phaseStepTarget = ofMap(ofGetMouseY(), 0, ofGetHeight(), phaseSpeed, 0, true);
  phaseStep = ofLerp(phaseStep, phaseStepTarget, 0.95);
}

void ofApp::draw()
{
  ofSetLineWidth(2.0);

  // Draw the left channel history.
  ofSetColor(200, 0, 0);
  ofBeginShape();
  for (int i = 0; i < leftSamples.size(); i++)
  {
    float x = ofMap(i, 0, leftSamples.size(), 0, ofGetWidth());
    float y = ofMap(leftSamples[i], -1.0, 1.0, 0, 200);
    ofVertex(x, y);
  }
  ofEndShape(false);
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, 0, ofGetWidth(), 200);

  // Draw the right channel history.
  ofSetColor(200, 0, 0);
  ofBeginShape();
  for (int i = 0; i < rightSamples.size(); i++)
  {
    float x = ofMap(i, 0, rightSamples.size(), 0, ofGetWidth());
    float y = ofMap(rightSamples[i], -1.0, 1.0, 200, 400);
    ofVertex(x, y);
  }
  ofEndShape(false);
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, 200, ofGetWidth(), 200);

  guiPanel.draw();
}

void ofApp::audioOut(ofSoundBuffer& buffer)
{
  float leftScale = 1 - pan;
  float rightScale = pan;

  // sin(n) seems to have trouble when n is very large.
  // Keep phase in the range of 0-TWO_PI.
  while (phaseVal > TWO_PI)
  {
    phaseVal -= TWO_PI;
  }

  for (int i = 0; i < buffer.getNumFrames(); i++)
  {
    phaseVal += phaseStep;
    float sample = sin(phaseVal);
    leftSamples[i] = buffer[i * 2 + 0] = sample * volume * leftScale;
    rightSamples[i] = buffer[i * 2 + 1] = sample * volume * rightScale;
  }
}
```

{{< image src="sound-sin.png" alt="Sound Sin" width="600px" >}}

### Noise Generator

We can generate signal from a variety of sources. Using [Perlin noise](https://en.wikipedia.org/wiki/Perlin_noise) leads to interesting results, because it offers a good balance between randomness and continuity. In OF, we can use [`ofSignedNoise()`](https://openframeworks.cc/documentation/math/ofMath/#show_ofSignedNoise) because it returns values in the same domain that our buffer expects, from `-1` to `1`.

```cpp
//ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"

class ofApp : public ofBaseApp 
{
public:
  void setup();
  void update();
  void draw();

  void audioOut(ofSoundBuffer& buffer);

  ofSoundStream soundStream;

  std::vector<float> leftSamples;
  std::vector<float> rightSamples;

  float phaseVal;
  float phaseStep;

  ofParameter<float> volume;
  ofParameter<float> pan;
  ofParameter<float> phaseSpeed;
  ofParameter<bool> doNoise;

  ofxPanel guiPanel;
};
```

```cpp
//ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofBackground(0);

  ofSoundStreamSettings settings;
  //settings.setApi(ofSoundDevice::Api::MS_DS);
  settings.setOutListener(this);
  //settings.sampleRate = 44100;
  settings.bufferSize = 256;
  settings.numOutputChannels = 2;
  settings.numInputChannels = 0;
  soundStream.setup(settings);

  phaseVal = 0;
  phaseStep = 0.0f;

  leftSamples.assign(settings.bufferSize, 0.0);
  rightSamples.assign(settings.bufferSize, 0.0);

  volume.set("Volume", 0.1, 0.0, 1.0);
  pan.set("Pan", 0.1, 0.0, 1.0);
  phaseSpeed.set("Phase Speed", 0.15, 0.0, 1.0);
  doNoise.set("Do Noise", false);

  guiPanel.setup("Audio Out", "settings.json");
  guiPanel.add(volume);
  guiPanel.add(pan);
  guiPanel.add(phaseSpeed);
  guiPanel.add(doNoise);
}

void ofApp::update()
{
  pan = ofMap(ofGetMouseX(), 0, ofGetWidth(), 0, 1, true);
  float phaseStepTarget = ofMap(ofGetMouseY(), 0, ofGetHeight(), phaseSpeed, 0, true);
  phaseStep = ofLerp(phaseStep, phaseStepTarget, 0.95);
}

void ofApp::draw()
{
  ofSetLineWidth(2.0);

  // Draw the left channel history.
  ofSetColor(200, 0, 0);
  ofBeginShape();
  for (int i = 0; i < leftSamples.size(); i++)
  {
    float x = ofMap(i, 0, leftSamples.size(), 0, ofGetWidth());
    float y = ofMap(leftSamples[i], -1.0, 1.0, 0, 200);
    ofVertex(x, y);
  }
  ofEndShape(false);
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, 0, ofGetWidth(), 200);

  // Draw the right channel history.
  ofSetColor(200, 0, 0);
  ofBeginShape();
  for (int i = 0; i < rightSamples.size(); i++)
  {
    float x = ofMap(i, 0, rightSamples.size(), 0, ofGetWidth());
    float y = ofMap(rightSamples[i], -1.0, 1.0, 200, 400);
    ofVertex(x, y);
  }
  ofEndShape(false);
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, 200, ofGetWidth(), 200);

  guiPanel.draw();
}

void ofApp::audioOut(ofSoundBuffer& buffer)
{
  float leftScale = 1 - pan;
  float rightScale = pan;

  // sin(n) seems to have trouble when n is very large.
  // Keep phase in the range of 0-TWO_PI.
  while (phaseVal > TWO_PI)
  {
    phaseVal -= TWO_PI;
  }

  if (doNoise)
  {
    for (int i = 0; i < buffer.getNumFrames(); i++)
    {
      phaseVal += phaseStep;
      float sample = ofSignedNoise(phaseVal);
      leftSamples[i] = buffer[i * 2 + 0] = sample * volume * leftScale;
      rightSamples[i] = buffer[i * 2 + 1] = sample * volume * rightScale;
    }
  }
  else
  {
    for (int i = 0; i < buffer.getNumFrames(); i++)
    {
      phaseVal += phaseStep;
      float sample = sin(phaseVal);
      leftSamples[i] = buffer[i * 2 + 0] = sample * volume * leftScale;
      rightSamples[i] = buffer[i * 2 + 1] = sample * volume * rightScale;
    }
  }
}
```

{{< image src="sound-noise.png" alt="Sound Noise" width="600px" >}}

## MIDI

MIDI (which stands for Musical Instrument Digital Interface) is a communication protocol that is a standard for audio messages. MIDI has been around since the 1980s and is pretty universal; if you have any piece of hardware or software that is used for music, there is a good chance it supports MIDI.

The MIDI specification consists of different types of messages, including:

* *Note On* and *Note Off* to trigger notes.
* *Polyphonic Key Pressure* and *Pitch Bend* to affect the played notes.
* *Control Change* and *Program Change* to set mode and functionality.
* *System* to control playback and device specific features.

In OF, we can use [`ofxMidi`](https://github.com/danomatika/ofxMidi) for input and output.

The following example uses a controller to trigger notes in OF.

* `ofApp` extends `ofxMidiListener` (on top of `ofBaseApp`) to get access to the MIDI listeners.
* `ofApp` implements the `newMidiMessage()` function to handle incoming messages.
* All possible frequencies are calculated at startup and stored in a look-up table.
* The target frequency is set whenever a MIDI note message is received.

```cpp
//ofApp.h
#pragma once

#include "ofMain.h"

#include "ofxGui.h"
#include "ofxMidi.h"

class ofApp : public ofBaseApp, public ofxMidiListener
{
public:
  void setup();
  void update();
  void draw();

  void updateWaveform(int& resolution);
  void audioOut(ofSoundBuffer& buffer);
  void newMidiMessage(ofxMidiMessage& msg);

  ofSoundStream soundStream;

  std::vector<float> leftSamples;
  std::vector<float> rightSamples;

  ofxMidiIn midiIn;
  std::vector<float> midiFreqs;
  float freqTarget;

  std::vector<float> waveform;
  float phaseVal;
  float phaseStep;

  ofParameter<float> volume;
  ofParameter<float> pan;
  ofParameter<int> resolution;
  ofParameter<float> freqSpeed;
  ofParameter<float> frequency;

  ofxPanel guiPanel;
};
```

```cpp
//ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  ofBackground(0);

  // Setup sound output.
  ofSoundStreamSettings settings;
  //settings.setApi(ofSoundDevice::Api::MS_DS);
  settings.setOutListener(this);
  //settings.sampleRate = 44100;
  settings.bufferSize = 256;
  settings.numOutputChannels = 2;
  settings.numInputChannels = 0;
  soundStream.setup(settings);

  phaseVal = 0;
  phaseStep = 0.0f;

  leftSamples.assign(settings.bufferSize, 0.0);
  rightSamples.assign(settings.bufferSize, 0.0);

  // Setup MIDI input.
  midiIn.listInPorts();
  midiIn.openPort(0);
  midiIn.ignoreTypes(true, true, true);
  midiIn.addListener(this);

  // Build note to frequency table.
  // http://subsynth.sourceforge.net/midinote2freq.html
  midiFreqs.resize(127);
  int a = 440; // a is 440 hz...
  for (int i = 0; i < midiFreqs.size(); i++)
  {
    midiFreqs[i] = (a / 32.0f) * powf(2, (i - 9) / 12.0f);
  }

  volume.set("Volume", 0.1, 0.0, 1.0);
  pan.set("Pan", 0.1, 0.0, 1.0);
  resolution.set("Resolution", 32, 3, 64);
  resolution.addListener(this, &ofApp::updateWaveform);
  freqSpeed.set("Freq Speed", 0.2, 0.0, 1.0);
  frequency.set("Frequency", 0.0, 0.0, 12544.0);

  guiPanel.setup("Audio Out", "settings.json");
  guiPanel.add(volume);
  guiPanel.add(pan);
  guiPanel.add(resolution);
  guiPanel.add(freqSpeed);
  guiPanel.add(frequency);

  // Set initial params.
  int res = resolution;
  updateWaveform(res);
}

void ofApp::update()
{
  pan = ofMap(ofGetMouseX(), 0, ofGetWidth(), 0, 1, true);
  frequency = ofLerp(frequency, freqTarget, freqSpeed);
}

void ofApp::draw()
{
  ofSetLineWidth(2.0);

  // Draw the left channel history.
  ofSetColor(200, 0, 0);
  ofBeginShape();
  for (int i = 0; i < leftSamples.size(); i++)
  {
    float x = ofMap(i, 0, leftSamples.size(), 0, ofGetWidth());
    float y = ofMap(leftSamples[i], -1.0, 1.0, 0, 200);
    ofVertex(x, y);
  }
  ofEndShape(false);
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, 0, ofGetWidth(), 200);

  // Draw the right channel history.
  ofSetColor(200, 0, 0);
  ofBeginShape();
  for (int i = 0; i < rightSamples.size(); i++)
  {
    float x = ofMap(i, 0, rightSamples.size(), 0, ofGetWidth());
    float y = ofMap(rightSamples[i], -1.0, 1.0, 200, 400);
    ofVertex(x, y);
  }
  ofEndShape(false);
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, 200, ofGetWidth(), 200);

  // Draw the waveform.
  ofSetColor(0, 200, 0);
  ofBeginShape();
  for (int i = 0; i < waveform.size(); i++)
  {
    float x = ofMap(i, 0, waveform.size() - 1, 0, ofGetWidth());
    float y = ofMap(waveform[i], -1.0, 1.0, ofGetHeight() - 120, ofGetHeight());
    ofVertex(x, y);
  }
  ofEndShape(false);
  ofSetColor(225);
  ofNoFill();
  ofDrawRectangle(0, ofGetHeight() - 120, ofGetWidth(), 120);

  guiPanel.draw();
}

// https://openframeworks.cc/ofBook/chapters/sound.html
void ofApp::updateWaveform(int& resolution)
{
  waveform.resize(resolution);

  // "waveformStep" maps a full oscillation of sin() to the size 
  // of the waveform lookup table
  float waveformStep = TWO_PI / (float)waveform.size();

  for (int i = 0; i < waveform.size(); i++)
  {
    waveform[i] = sin(i * waveformStep);
  }
}

void ofApp::audioOut(ofSoundBuffer& buffer)
{
  float leftScale = 1 - pan;
  float rightScale = pan;

  // sin(n) seems to have trouble when n is very large.
  // Keep phase in the range of 0-TWO_PI.
  while (phaseVal > TWO_PI)
  {
    phaseVal -= TWO_PI;
  }

  float sampleRate = buffer.getSampleRate();
  phaseStep = frequency / sampleRate;

  for (int i = 0; i < buffer.getNumFrames(); i++)
  {
    phaseVal += phaseStep;
    int idx = (int)(phaseVal * waveform.size()) % waveform.size();
    float sample = waveform[idx];
    leftSamples[i] = buffer[i * 2 + 0] = sample * volume * leftScale;
    rightSamples[i] = buffer[i * 2 + 1] = sample * volume * rightScale;
  }
}

void ofApp::newMidiMessage(ofxMidiMessage& msg)
{
  ofLogNotice(__FUNCTION__) << msg.getStatusString(msg.status) << " Pitch: " << msg.pitch << " Velocity: " << msg.velocity << " Control: " << msg.control << " Value: " << msg.value;
  if (msg.status == MidiStatus::MIDI_NOTE_ON)
  {
    freqTarget = midiFreqs[msg.pitch];
  }
  else if (msg.status == MidiStatus::MIDI_NOTE_OFF)
  {
    freqTarget = 0;
  }
}
```

## Sound Addons

There are a few other sound addons available to explore. Some recommendations:

* [ofxSoundObjects](https://github.com/roymacdonald/ofxSoundObjects) uses the idea of *sound objects* as components that audio moves through. Each has an input and output, so they can be connected to one another to stack sounds and filters.
* [ofxTonic](https://github.com/TonicAudio/ofxTonic) is an OF wrapper for [Tonic](https://github.com/TonicAudio/Tonic), a high performance C++ audio synthesis library. It works on top of the OF sound engine and makes sound generation fun and easy.
