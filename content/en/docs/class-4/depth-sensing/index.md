---
title: "Depth Sensing"
description: ""
lead: ""
date: 2022-10-07T11:53:02-04:00
lastmod: 2022-10-07T11:53:02-04:00
draft: true
images: []
menu:
  docs:
    parent: "class-4"
    identifier: "depth-sensing"
weight: 520
toc: true
---

Most of the apps we have built up to now are using video as input. Video is two-dimensional, it has a width and a height.

Depth sensors are special types of cameras that add a third dimension to the mix; they can see width, height, and *depth*. Depth in this context means the distance away from the camera where a captured pixel is situated.

It is important to note the difference between full 3D and depth. Just like a conventional 2D camera, a depth sensor can only capture what it "sees". If an object is obstructing another one behind it, the camera will only be able to measure the depth of the one closest to it. We can only have one depth measurement per pixel.

{{< image src="pin-art.jpg" alt="Metal Pin Art Pin Point Impression 3D Frame Toy" caption="[*Metal Pin Art Pin Point Impression 3D Frame Toy*](https://www.fasttech.com/product/1162800-metal-pin-art-pin-point-impression-3d-frame-toy)" width="600px" >}}

<figure style="width:600px;height:400px;display:block;margin:0 auto;">
<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/86175057?color=ffffff&title=0&byline=0&portrait=0" style="position:absolute;top:0;left:0;width:100%;height:100%;" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>
<figcaption><i><a href="https://vimeo.com/86175057">MegaFaces: Kinetic Facade Shows Giant 3D &#039;Selfies&#039;</a> from <a href="https://vimeo.com/iart">iart</a> on <a href="https://vimeo.com">Vimeo</a>.</i></figcaption>
</figure>

## Technologies

There are a few different technologies used for depth sensing, each with their own pros and cons depending on the application.

Most depth cameras include both color and depth sensors, providiing a pair of images per frame. These are then combined to form a colored point cloud as the result.

### Coded (Structured) Light

[Coded (or structured) light](https://en.wikipedia.org/wiki/Structured-light_3D_scanner) works by projecting a known pattern onto the world, then analyzing the deformations of the pattern to determine the shapes of the surfaces it is projected on.

<figure style="width:600px;display:block;margin:0 auto;">
<iframe width="560" height="315" src="https://www.youtube.com/embed/PluL7WTlKrM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<figcaption><i><a href="https://www.youtube.com/embed/PluL7WTlKrM">Coded light technology and the Intel­® RealSense™ Depth Camera SR305</a></i></figcaption>
</figure>

* Pros:
  * Precise.
  * High resolution, usually as high as the camera resolution it is using.
* Cons:
  * Can be slow as it needs multiple patterns projected then captured to generate a single frame.
  * Subject to interference from outside light, so usually better for indoor sensing.

### Time of Flight

Like the name implies, [time of flight](https://en.wikipedia.org/wiki/Time-of-flight_camera) works by measuring the time it takes for a beam of light to do a round-trip to the sensor. This involves complex calculations using the speed of light, and the shorter the travel time, the nearer the object.

{{< image src="https://www.stemmer-imaging.com/media/cache/default_image_dialog/uploads/cameras/sis/gl/glossary-3D-time-of-flight-cameras-1-en.jpg" alt="3D time of flight cameras" caption="[*3D time of flight cameras*](https://www.stemmer-imaging.com/en/knowledge-base/cameras-3d-time-of-flight-cameras/)" width="600px" >}}

* Pros:
  * Compact. (This is the technology used in many current generation phones)
  * Fast, and ideal for real-time processing.
* Cons:
  * Mid-level accuracy.
  * Low resolution as it uses a custom sensor.
  * Subject to interference from other devices.

### Stereo Vision

Stereo vision works like human depth perception, where two cameras (or eyes) are placed side by side looking in the same direction. Differences in the two captured images, called *disparity*, is used to determine how far from the sensors these different pixels are.

{{< image src="https://upload.wikimedia.org/wikipedia/commons/f/fd/Sgm_method.PNG" alt="Semi-global Matching Method" caption="[*Semi-global Matching Method*](https://commons.wikimedia.org/wiki/File:Sgm_method.PNG)" width="600px" >}}

* Pros:
  * Tends to be cheap, as any off the shelf cameras can be used.
  * Image representation is intuitive to humans.
  * No interference.
* Cons:
  * Low accuracy.
  * Complex implementation requiring feature extraction and matching.

## Microsoft Kinect

Microsoft was the first company to bring depth sensors to the mass market with the Kinect for Xbox 360 in 2010. This was originally designed as a game controller, but gained a lot of interest from robotics engineers and creative coders, who [hacked the device](https://www.wired.com/2011/06/mf_kinect/) to use for custom Desktop applications.

Microsoft originally was against the practice, even threatening legal action against the hackers, but eventually changed course and embraced the community building around it.

<figure style="width:600px;display:block;margin:0 auto;">
<div style="padding:62.5% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/16818988" style="position:absolute;top:0;left:0;width:100%;height:100%;" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>
<figcaption><i><a href="https://vimeo.com/16818988">ofxKinect 3D draw 001</a> from <a href="https://vimeo.com/memotv">Memo Akten</a> on <a href="https://vimeo.com">Vimeo</a>.</i></figcaption>
</figure>

<figure style="width:600px;display:block;margin:0 auto;">
<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/16985224" style="position:absolute;top:0;left:0;width:100%;height:100%;" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>
<figcaption><i><a href="https://vimeo.com/16985224">Interactive Puppet Prototype with Xbox Kinect</a> from <a href="https://vimeo.com/muonics">Theo Watson</a> on <a href="https://vimeo.com">Vimeo</a>.</i></figcaption>
</figure>

### Kinect for Xbox 360

Even though it was discontinued in 2013, this device is still used because of its long range and ease of use on all major OSes.

{{< image src="kinect_v1.png" alt="Kinect for Xbox 360" width="600px" >}}

* Technology: Coded light
* Range: `1.2-3.5m` / `3.9-11.5ft`
* Color resolution: `640x480px`
* Depth resolution: `640x480px`
* FOV: `57°x43°`

Extra features:

* Microphone
* Motorized base to adjust device angle

OF Support

* [ofxKinect](https://openframeworks.cc/documentation/ofxKinect/)
  * Windows, Mac, Linux
  * [README](https://github.com/openframeworks/openFrameworks/tree/master/addons/ofxKinect) for setup instructions
  * Based on [libfreenect](https://openkinect.org/wiki/Main_Page)

<figure style="width:600px;display:block;margin:0 auto;">
<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/36892768?color=ffffff&badge=0" style="position:absolute;top:0;left:0;width:100%;height:100%;" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>
<figcaption><i><a href="https://vimeo.com/36892768">Starfield</a> from <a href="https://vimeo.com/lab212">Lab212</a> on <a href="https://vimeo.com">Vimeo</a>.</i></figcaption>
</figure>

### Kinect V2 (aka Kinect for Xbox One)

Released in 2012, the second version of the Kinect is said to have 3x the fidelity of its predecessor and a 60% wider field of view.

While still originally labeled as an Xbox controller, Microsoft shipped a Windows SDK with the device, providing access to advanced features to Desktop applications.

This device was discontinued in 2017 but is also still widely used for long-term installations.

{{< image src="kinect_v2.jpg" alt="Kinect V2" width="600px" >}}

* Technology: Time-of-flight
* Range: `0.5-4.5m` / `1.6-11.5ft`
* Color resolution: `1920x1080px`
* Depth resolution: `512x424px`
* FOV: `70°x60°`

Extra features using [Microsoft SDK](https://developer.microsoft.com/en-us/windows/kinect):

* Body tracking (up to 6 people)
* Facial expression recognition
* Hand gesture recognition
* Heart rate tracking
* Speech recognition

OF Support

* [ofxKinectV2](https://github.com/ofTheo/ofxKinectV2)
  * Windows, Mac, Linux
  * Based on [libfreenect2](https://github.com/OpenKinect/libfreenect2)
  * Note that this does not support any Microsoft SDK features like Body Tracking
* [ofxKinectForWindows2](https://github.com/elliotwoods/ofxKinectForWindows2)
  * Windows only!
  * Requires [Microsoft SDK](https://developer.microsoft.com/en-us/windows/kinect)

<figure style="width:600px;display:block;margin:0 auto;">
<div style="padding:56.25% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/96615251?title=0&byline=0&portrait=0" style="position:absolute;top:0;left:0;width:100%;height:100%;" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>
<figcaption><i><a href="https://vimeo.com/96615251">Parade - Dancing Shadow Sculptures</a> from <a href="https://vimeo.com/bonjourdpt">Dpt.</a> on <a href="https://vimeo.com">Vimeo</a>.</i></figcaption>
</figure>

### Kinect for Azure

Just released in 2019, the [Kinect for Azure](https://azure.microsoft.com/en-us/services/kinect-dk/) is based on technologies of the previous Kinect and the [HoloLens](https://www.microsoft.com/en-us/hololens). This is the first Kinect device marketed to developers at launch, and the first device to have an [open-source SDK](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK) with official support for non-Windows platforms.

{{< image src="kinect_v3.png" alt="Kinect for Azure" width="600px" >}}

* Technology: Time-of-flight
* Range: `0.25-5.5m` / `0.8-11.5ft` (mode dependent)
* Color resolution: Up to `3840x2160px`
* Depth resolution: Up to `1024x1024px`
* FOV: Up to `120°x120°`

Extra features using [Microsoft SDK](https://developer.microsoft.com/en-us/windows/kinect):

* Orientation sensors
* 360° microphone array
* Body tracking (requires NVIDIA GPU)

OF Support

* [ofxAzureKinect](https://github.com/prisonerjohn/ofxAzureKinect)
  * Windows, Linux
  * Requires [Azure Kinect Sensor SDK](https://docs.microsoft.com/en-us/azure/kinect-dk/sensor-sdk-download)

## Intel RealSense

The [Intel RealSense](https://www.intelrealsense.com/) started off as a compact depth camera aimed at video conferencing, gesture-based interaction, and 3D scanning. The 200 series included many devices, both standalone and embedded in tablet computers and laptops.

The quality was not up-to-par with other depth cameras, and the original RealSense was rarely used for interactive installations.

In 2018, Intel released the 400 series devices, which were a major improvement on the previous generation. Low cost, small form-factor and portability make these devices a viable choice for many applications.

{{< image src="realsense_d400_cameras.jpg" alt="Intel RealSense" width="600px" >}}

### D415 / D435 / D455 / D457

* Technology: Stereo IR
* Range: `0.10-10m` / `0.3-32ft` (depends on conditions)
* Color resolution: `1920x1080px`
* Depth resolution: `1280x720px`
* FOV: `65°x40°` (D415), `87°x58°` (D435, D455, D457)

Extra features:

* USB powered
* Orientation sensors (on some models)

OF Support

* [ofxLibRealSense2](https://github.com/m9dfukc/ofxLibRealSense2)
  * Mac (Windows, Linux in theory)
* [ofxRealSense2](https://github.com/prisonerjohn/ofxRealSense2)
  * Windows (Mac, Linux in theory)

## Other Options

### Stereolabs ZED

Stereolabs are the newest addition to this list, and are worth mentioning because of their high quality [ZED 2](https://www.stereolabs.com/zed-2/) sensors.

* Technology: Stereo Color (range is virtually unlimited)
* Waterproof / dustproof options makes them great for outdoor use
* SDK uses machine learning models to provide a robust depth map and body tracking features
* Works on Windows and Linux but requires an NVIDIA GPU

### Orbbec Astra

Orbbec released the [Astra Series](https://orbbec3d.com/index/Product/info.html?cate=38&id=36) as a response to the Microsoft Kinect. The goal was to create an open, cross-platform SDK which included body tracking with OpenNI.

* Technology: Strucutured Light

### Leap Motion

The [Leap Motion Controller](https://www.ultraleap.com/product/leap-motion-controller/) is a depth sensor that focuses on hand and finger tracking. It can be used for both Desktop and VR applications.

* Technology: Stereo IR

## USB Connections

We will often find ourselves wanting to connect many sensors to a single computer, or wanting to position our sensors far from the computer. This can be achieved using USB hubs and USB extenders, but one thing to remember is that not all USBs are created equal, and like most things in life, you get what you pay for.

In all cases, the most important thing we can do is test our setup with all the hardware connected to make sure everything is working as expected.

### Bandwidth

USB bandwidth (amount of data over time) should be planned carefully:

* Only enable feeds that are necessary to the application (e.g. Disable the RGB color stream if we are only interested in the depth data).
* Use a lower image resolution if it provides enough information (e.g. Test a lower resolution stream to see if it provides enough precision).
* If using multiple devices, connect them to different USB channels when possible (e.g. Connect one on the front and the other on the back).

### Hubs

USB hubs can help, but have limitations:

* Powered USB hubs will have higher bandwidth as they can use more power.
* External hubs that connect to a PC using a USB connection will create a bottleneck (since all the data still needs to go through a single bus).
* Internal PCIe hubs with dedicated channels per port will work the best.

Recommendation:

* [StarTech PCI Express Adapter Card](https://www.startech.com/en-us/cards-adapters/pexusb3s44v)

### Cables

USB cables will deteriorate the signal over distance.

* Use cables that are close in length to what is needed. Cables that are too long will weaken the signal.
* Thicker and insulated cables tend to reduce interference.
* Active (powered) cables can boost the data signal, especially over long distances.

Recommendations:

* [Cable Matters Active USB 3.0 extension cable](https://www.cablematters.com/pc-512-135-active-usb-30-extension-cable-usb-3-extension-cable-usb-extension-cable-male-to-female.aspx)
* [Newnex active cables](https://www.newnex.com/usb-type-c-cables-legacy.php)
