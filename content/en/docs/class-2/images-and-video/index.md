---
title: "Images and Video"
description: ""
lead: ""
date: 2022-09-18T13:42:06-04:00
lastmod: 2022-09-18T13:42:06-04:00
draft: false
images: []
menu:
  docs:
    parent: "class-2"
    identifier: "images-and-video"
weight: 230
toc: true
---

## File Formats

Digital images come in a variety of formats, each with their own properties.

### Vector Graphics

*Vector* formats define a set of points and instructions on how to draw them. The instructions are run by a program to raster the image in order to view it.

Some of the more common vector formats are `SVG`, `EPS`, `PDF`, and `AI`.

If we open the following `SVG` file in a text editor, we will notice that it is fairly easy to read the format. It almost reads like a Processing program 😉

{{< image src="shapes.svg" alt="Shapes SVG" align="center" >}}

```svg
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<svg
   ...
   height="512"
   width="512">
  ...
  <g
     transform="translate(0,-161.53332)"
     id="layer1">
    <circle
       style="stroke-width:0.26458332;fill:#00ffff;fill-opacity:1"
       r="52.916664"
       cy="229.26665"
       cx="67.73333"
       id="path3713" />
    <rect
       y="228.20831"
       x="5.2916665"
       height="63.5"
       width="63.5"
       id="rect4520"
       style="fill:#ff0000;fill-opacity:1;stroke-width:0.25843021" />
    <path
       id="path4524"
       d="M 49.514879,171.88985 123.5982,282.2589 Z"
       style="fill:none;stroke:#00b400;stroke-width:2.64583325;stroke-linecap:butt;stroke-linejoin:miter;stroke-miterlimit:4;stroke-dasharray:none;stroke-opacity:1" />
  </g>
</svg>
```

Pros:

* Small file sizes, because minimal information is being stored.
* Images can be scaled up without any quality loss or increase in file size. This is because the instruction set does not change, the only thing that changes is the point values.

Cons:

* Low level of detail.
* Limited types of effects, because we don't have all the image data available in the format.

### Raster Graphics

*Raster* formats define pixel values in a rectangular grid of pixels. The bigger the image, the greater the data set, and thus the larger the file size.

Some of the more common vector formats are `JPG`, `PNG`, `GIF`, and `TIF`.

{{< image src="shapes.png" alt="Shapes PNG" align="center" >}}

Pros:

* High quality and detail, especially at high resolutions.
* More advanced image effects, because every pixel can be edited.

Cons:

* File sizes tend to be bigger.
* Images lose quality when scaled up.

In order not to end up with huge file sizes, many raster formats are compressed. Some compression methods are *lossy*, meaning that some of the data is lost when it is compressed, and others are *lossless*, meaning that all the data is recovered once the data is uncompressed.

### Video

Videos are just a series of images that need to be processed and displayed very quickly.

Video formats are always rasters and are mostly compressed.

* Some formats are simply extensions of their image counterparts, like `Motion JPG` for example, which is just a series of `JPG`-compressed frames.
* Others are specific to video, like `H.264`, which has a form of compression over time, where some pixels are predicted based on known pixels in previous and future key frames. This is called *temporal compression*.

Efficient compression is necessary for video because of the huge amount of data that it carries. While film used to run at 24 frames per second, high definition video now runs standard at 60 frames per second, and sometimes goes as high as 240 fps! Combining these fast frame rates with large resolutions like 4K means that hundreds of millions of pixels need to be processed every second for a video to play smoothly.

### Processing Images

When working with image data, we will usually want to work with rasterized uncompressed images. This is because many algorithms require looping efficiently through all pixels in an image, or doing quick look-ups between neighboring pixels.

The good news is that this usually happens in the image loader or video codec, before an image or video frame gets to us. For example in OF, FreeImage will automatically decompress `JPG` or `PNG` images and provide us the "final" pixels in the frame.

While we will almost never have to worry about decoding an image or a video frame ourselves, we should still be mindful of what format the data comes in, and make sure that it is suitable for our application.

## Images in OF

### The `data` Folder

The simplest way to access files in an OF app is to include them in the project's `data` folder. If this looks familiar, it's because this idea is borrowed from Processing. The `data` folder is located in `<project>/bin/data` and each project will have its own dedicated `data` folder.

If we drop our files in the `data` folder, they can be accessed in the app without having to figure out the full path on disk where the file is located, which can be very handy.

### `ofImage`

[`ofImage`](https://openframeworks.cc/documentation/graphics/ofImage/) is the general type to use to work with images in openFrameworks. `ofImage` includes methods to load files from disk, draw images to the screen, access pixel data, etc.

`ofImage` is a type, which we can create like any other variable type.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void draw();

  ofImage dogImg;
};
```

{{< alert context="info" icon="✌️" >}}
**Declaring variables in the header file**

Variables we want to use in any method (function) of our class should be declared in the header. This means that they are in the entire class' *scope*.

If we declare a variable inside one of our methods like `ofApp::setup()` or `ofApp::draw()`, that variable will only be part of that specifc method's scope, and will not be accessible outside of it.
{{< /alert >}}

We will load an image named [`dog-grass.jpg`](dog-grass.jpg) from our `data` folder in the `ofApp::setup()` function. We only need to load the image into memory once, so we do it when the app starts up.

We want to draw the image every frame, so we will do that in the `ofApp::draw()` function.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  dogImg.load("dog-grass.jpg");
}

void ofApp::draw()
{
  dogImg.draw(0, 0);
}
```

{{< alert context="info" icon="✌️" >}}
**What happened to the other built-in ofApp methods?**

In this specific example, we are not using most of the `ofApp` placeholder methods like `ofApp::update()`, `ofApp::keyPressed()`, `ofApp::mouseMoved()`, etc. We can remove them from our class if we will not be using them, and this will increase readability because the code will not be filled with stub functions.

However, note that the method needs to be removed from both the header `.h` and the implementation `.cpp` files or else the compiler will assume something is missing and will throw an error.
{{< /alert >}}

If we navigate under the hood and see what `ofImage::load()` is actually doing, we see that it calls many functions from the `FreeImage` library to determine the file's format, uncompress the data, and load it into values for each pixel.

## Image Attributes

An image data structure usually comprises of:

* a size (a width and height)
* a pixel format
* a value for each pixel

### Pixel Arrays

This structure looks a lot like the arrays we have been exploring in the previous section. This makes arrays great options to represent image data in a computer program.

Even though an image has two dimensions (a width and a height), the pixel array is usually one-dimensional, packing the rows one after the other in sequence.

{{< image src="grid-pix.png" alt="Grid Pixels" align="center" >}}

Some frameworks allow accessing pixels using the column `x` and row `y`, like [`PImage.get()`](https://www.processing.org/reference/PImage_get_.html) in Processing and [`ofImage.getColor()`](https://openframeworks.cc//documentation/graphics/ofImage/#!show_getColor) in openFrameworks. These convenience functions are very useful as they take care of figuring out all the index arithmetic for us.

The following example draws an image one pixel at a time, using nested for-loops to iterate through each row and column.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  // Load the dog image.
  dogImg.load("dog-grass.jpg");

  // Set the window size to match the image.
  ofSetWindowShape(dogImg.getWidth(), dogImg.getHeight());
}

void ofApp::draw()
{
  for (int y = 0; y < dogImg.getHeight(); y++)
  {
    for (int x = 0; x < dogImg.getWidth(); x++)
    {
      ofColor color = dogImg.getColor(x, y);
      ofSetColor(color);
      ofDrawRectangle(x, y, 1, 1);
    }
  }
}
```

* [`ofSetWindowShape()`](https://openframeworks.cc/documentation/application/ofAppRunner/#show_ofSetWindowShape) resizes the window to the size of the loaded image. Note that this function can be called any time while the app is running, and can override the starting window dimensions that are set in `main.cpp`.
* [`ofImage.getColor()`](https://openframeworks.cc/documentation/graphics/ofImage/#!show_getColor) returns the [`ofColor`](https://openframeworks.cc/documentation/types/ofColor/) value at a specified column and row index. `ofColor` is a data structure used to access the different channels that make up a color value.

{{< details "How would we read the value  of a pixel under the mouse cursor?" >}}

We can use `ofImage.getColor()` and pass the mouse coordinates as the column and row index.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  // Load the dog image.
  dogImg.load("dog-grass.jpg");

  // Set the window size to match the image.
  ofSetWindowShape(dogImg.getWidth(), dogImg.getHeight());
}

void ofApp::draw()
{
  // Draw the image as the background.
  ofSetColor(255);
  dogImg.draw(0, 0);

  // Get a reference to the image pixels.
  ofPixels dogPix = dogImg.getPixels();
  // Get the color value under the mouse.
  ofColor color = dogPix.getColor(mouseX, mouseY);

  // Draw a rectangle under the mouse using the pixel color.
  ofFill();
  ofSetColor(color);
  ofDrawRectangle(mouseX - 25, mouseY - 25, 50, 50);
  // Add an outline so we can see the rectangle better.
  ofNoFill();
  ofSetColor(0);
  ofDrawRectangle(mouseX - 25, mouseY - 25, 50, 50);
}
```

Note that this only works if the window and image have equal resolutions. If they didn't, we would need to remap the mouse coordinates to the window coordinates. We will cover this in a later class.

{{< /details >}}

### Size and Scale

In the previous examples, the drawn image is anchored in the top-left corner of the window `(0, 0)` and by default, it is drawn at full resolution. This means that the image might be smaller or larger than our window.

If we want the image to fill and fit in the exact window bounds, we have two options:

We can resize the window to match the image resolution. This is what we have been doing in the previous examples.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  // Load the dog image.
  dogImg.load("dog-grass.jpg");

  // Set the window size to match the image.
  ofSetWindowShape(dogImg.getWidth(), dogImg.getHeight());
}

void ofApp::draw()
{
  dogImg.draw(0, 0);
}
```

We can scale the image to match the window size.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  // Load the dog image.
  dogImg.load("dog-grass.jpg");

  ofSetWindowShape(1280, 720);
}

void ofApp::draw()
{
  dogImg.draw(0, 0, ofGetWidth(), ofGetHeight());
}
```

* If `ofImage::draw()` is called with 4 arguments, the first 2 set the top-left coordinates and the last 2 set the width and height.

{{< alert context="info" icon="✌️" >}}
**Pixels and Textures**

At this point, it's a good idea to have a basic understanding of how modern computers draw images to the screen.

Two different processing units are used for this to occur.

* A *pixel color array* lives on the *CPU*. We can load data into this array and read it and change it. In OF, this is represented by  [`ofPixels`](https://openframeworks.cc/documentation/graphics/ofPixels/).
* A *texture* lives on the *GPU*. It cannot be manipulated (easily) and is read to be rendered on a screen. In OF, this is represented by  [`ofTexture`](https://openframeworks.cc/documentation/graphics/ofTexture/).
* When the image is ready to be drawn, the pixel array is sent from the CPU to the GPU and converted to a texture.

`ofImage` contains an `ofPixels` and an `ofTexture` and will try to handle the CPU to GPU transfer automatically.
 {{< /alert >}}

If we use an [image that is smaller than the window](dog-grass-low.jpg), it will be scaled up to fit the window. We can tell OF how to upscale the image by setting a filter on the *texture*.

When an image is scaled up, it needs additional pixels to fill in the extra resolution. Conversely, when an image is scaled down, it removes some of its original pixels because the resolution is smaller. The *min* and *mag* filters define how the renderer should handle these situations.

* The default mode uses linear interpolation `GL_LINEAR`. This blends the nearby pixels together to make new pixels and may look blurry.
* The nearest neighbor mode `GL_NEAREST` uses the nearest pixel value for the added pixels without any blending. This keeps the image sharp at any resolution, but it may look pixelated.

```cpp
// ofApp.h
#pragma once

#include "ofMain.h"

class ofApp : public ofBaseApp
{
public:
  void setup();
  void draw();

  void mousePressed(int button, int x, int y);
  void mouseReleased(int button, int x, int y);

  ofImage dogImg;
};
```

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
  dogImg.load("dog-grass-low.jpg");

  ofSetWindowShape(1280, 720);
}

void ofApp::draw()
{
  dogImg.draw(0, 0, ofGetWidth(), ofGetHeight());
}

void ofApp::mousePressed(int button, int x, int y)
{
  dogImg.getTexture().setTextureMinMagFilter(GL_NEAREST, GL_NEAREST);
}

void ofApp::mouseReleased(int button, int x, int y)
{
  dogImg.getTexture().setTextureMinMagFilter(GL_LINEAR, GL_LINEAR);
}
```

* We can access the texture with [`ofImage.getTexture()`](https://openframeworks.cc/documentation/graphics/ofImage/#!show_getTexture) and set the *min* (scale down) and *mag* (scale up) filters with [`ofTexture.setTextureMinMagFilter()`](https://openframeworks.cc/documentation/graphics/ofTexture/#!show_setTextureMinMagFilter).
* Texture filters do not use any additional computation, because they are handled automatically by the GPU!

{{< image src="dog-linear.png" alt="Dog using GL_LINEAR" caption="Dog using <code>GL_LINEAR</code>" width="320px" >}}

{{< image src="dog-nearest.png" alt="Dog using GL_NEAREST" caption="Dog using <code>GL_NEAREST</code>" width="320px" >}}

### Pixel Access

A standard color pixel will have 3 color channels: red, green, and blue (`RGB`). While Processing packs all channels into a single `int`, this is not common practice.

The color values are usually packed *sequentially* in the array. Instead of each pixel holding a single value, it will hold 3.

{{< image src="grid-rgb.png" alt="Grid RGB" align="center" >}}

The pixel array then has total size:

```cpp
size = width * height * channels
```

In order to access the pixel in a 1D array using a 2D index, we first need to convert it.

 ```cpp
 index = y * width + x
 ```

How do we access a pixel index in an `RGB` image?

Because each pixel has three color values (for each `RGB` channel), we need to multiply our pixel index by `3` to take that offset into account.

```cpp
pixel = y * width + x

index = pixel * 3
index = (y * width + x) * 3
```

{{< details "<code>ofPixels.getColor()</code> can also accept a single argument for the index (instead of two arguments for the column and row). How can we modify the previous example to use the single index version of <code>getColor()</code>?" >}}

We can use the formula above to convert our column and row to an index value in the color array.

```cpp
// ofApp.cpp

// ...

void ofApp::draw()
{
  // ...

  // Get a reference to the image pixels.
  ofPixels dogPix = dogImg.getPixels();
  // Get the color value under the mouse.
  //ofColor color = dogPix.getColor(mouseX, mouseY);
  int index = (mouseY * dogPix.getWidth() + mouseX) * dogPix.getNumChannels();
  ofColor color = dogPix.getColor(index);

  // ...
}
```

Note the use of [`ofPixels.getNumChannels()`](https://openframeworks.cc//documentation/graphics/ofPixels/#!show_getNumChannels) instead of the literal `3`. This ensures the code will work with all image types and not just RGB images.
{{< /details >}}

Conversely, if we want to get a 2D value from a 1D index, we can use integer division:

```cpp
x = index % width
y = index / width
```

 The following example reads the value of a pixel sequentially, based on the sketch frame number.

```cpp
// ofApp.cpp
#include "ofApp.h"

void ofApp::setup()
{
    // Load the dog image.
    dogImg.load("dog-grass.jpg");

    // Set the window size to match the image.
    ofSetWindowShape(dogImg.getWidth(), dogImg.getHeight());
}

void ofApp::draw()
{
  // Draw the image as the background.
  ofSetColor(255);
  dogImg.draw(0, 0);

  // Cache the image dimensions in variables for easy access.
  int imgWidth = dogImg.getWidth();
  int imgHeight = dogImg.getHeight();

  // Use the modulo operator to make sure the frame index is never 
  // greater than the max number of pixels in the image.
  int frameIndex = ofGetFrameNum() % (imgWidth * imgHeight);
  int x = frameIndex % imgWidth;
  int y = frameIndex / imgWidth;

  // Get a reference to the image pixels.
  ofPixels dogPix = dogImg.getPixels();
  // Get the color value for this frame.
  int pixelIndex = frameIndex * dogPix.getNumChannels();
  ofColor color = dogPix.getColor(pixelIndex);

  // Draw a rectangle under the mouse using the pixel color.
  ofFill();
  ofSetColor(color);
  ofDrawRectangle(x - 25, y - 25, 50, 50);
  // Add an outline so we can see the rectangle better.
  ofNoFill();
  ofSetColor(0);
  ofDrawRectangle(x - 25, y - 25, 50, 50);
}
```

## Data Formats

### Image Format

The most common image type we will work with is `RGB` color images.

We will also work with single-channel formats, usually called grayscale or luminance. These are particularly handy for devices that only capture a brightness level, like infrared cameras or depth sensors.

Some images also have an alpha channel for transparency, like `RGBA`. Our example image happens to have transparency, but we will encounter this rarely in this class as most sensors do not use the alpha channel.

Another format worth mentioning is [`YUV`](https://en.wikipedia.org/wiki/YUV), which is a color encoding that is based on the range of human perception. Instead of using three channels for color, it uses one for brightness and two for color shift. This gives similar results to `RGB` but at much smaller sizes (usually a third), and this is why `YUV` formats are often used for webcam streams.

### Pixel Format

Pixel color values can be stored in a few different formats. The more bits a format can hold, the more range the values can have, and the larger the size of the frame gets.

* `unsigned char` is the most common format. It uses integers and each channel has 8 bits of data and values range from `0` to `255`.
* `float` uses floating point 32 bit data. The usual range is from `0.0` to `1.0` but this format can be used for [HDR](https://en.wikipedia.org/wiki/High-dynamic-range_imaging) effects, where the values can extend past `1.0` or for storing non-color data, where we can even use negative values. We will use `float` when working with depth sensors and when storing non-color data inside our pixels.
* `unsigned short` is another integer format but with 16 bits of data, meaning values range from `0` to `65535`. We will also use this format when working with depth sensors, where precision is very important and we need more than the `256` distinct values that we get from `unsigned char`.

The following example demonstrates how to access the pixel array data directly, using [`ofPixels.getData()`](https://openframeworks.cc/documentation/graphics/ofPixels/#show_getData).

This is a bit more complicated, and may not be necessary in most applications. However, it tends to be the fastest way to manipulate pixel values and is the recommended approach when having to process large images pixel by pixel.

```cpp
// ofApp.cpp

// ...

void ofApp::draw()
{
  // ...

  // Get a reference to the image pixels.
  unsigned char* dogData = dogImg.getPixels().getData();
  // Get the color value for this frame.
  int numChannels = dogImg.getPixels().getNumChannels();
  int pixelIndex = mouseY * dogImg.getWidth() + mouseX;
  ofColor color = ofColor(
    dogData[pixelIndex * numChannels + 0], // R
    dogData[pixelIndex * numChannels + 1], // G
    dogData[pixelIndex * numChannels + 2]  // B
  );

  // ...
}
```

{{< alert context="info" icon="✌️" >}}
**What does the <code>*</code> after <code>unsigned char</code> mean?**

The `*` represents something called a *pointer*. Pointers are a complex topic that we will cover in depth later in the course, but for now just think of them as representing arrays with a variable size (or arrays with a size we do not know at compile time).

The code above needs to work for any image of any size, so we cannot assume that the `unsigned char` array will have a specific number of elements in it. Using `unsigned char*` tells the compiler that the array size will be dynamically allocated when it is created.
{{< /alert >}}
