---
title: "Assignment 2"
description: ""
date: 2022-09-19T08:43:20-04:00
lastmod: 2022-09-19T08:43:20-04:00
draft: false
images: []
menu:
  docs:
    parent: "assignments"
    identifier: "assignment-2"
weight: 200
---

## Video Effect

### Instructions

Your assignment is to create an original video effect using openFrameworks and OpenCV. This will give you an opportunity to play with an image’s pixel data, and to get familiar with computer vision algorithms.

* You can use some of the algorithms we covered in class (background subtraction, color tracking, etc.) or any other CV algorithm. Make sure to browse through ofxCv examples to see what is available to you!
* If you have an idea but are not sure how to do it, ask about it on Discord and we can break it down and figure it out together.
* Expose your parameters and use a GUI or some form of controller to tweak them for best results.
* Some of you will demo your project in class. Your effect should therefore work in the conditions of our classroom (size, layout, light, etc.)

### Bonus Points!

If you are looking for an additional challenge, try adding the following extra features:

1. Press a key or GUI button to freeze / restart the video capture.
1. Press a key or GUI toggle to switch between live video ([`ofVideoGrabber`](https://openframeworks.cc/documentation/video/ofVideoGrabber/)) and on-disk video ([`ofVideoPlayer`](https://openframeworks.cc/documentation/video/ofVideoPlayer/)) as the input.

## Delivery

* Name your project `SM02-FirstLast` where **First** is your first name and **Last** is your last name.

```python
- OF/
  - apps/
    - seeing-machines/
      - SM02-ElieZananiri/
        - src/
        - bin/
        - addons.make
        - SM02-ElieZananiri.sln
        - SM02-ElieZananiri.vcxproj
        - ...
```

* Only submit the necessary files to rebuild your project.

  * This includes sources, the `addons.make` file, and any resources in your `data` folder.
  * No project or compiled files.
  * In the example above, you would only keep the `src` folder, `addons.make` file, and `bin/data` if you are using any external assets.
  * Zip the `SM02-ElieZananiri` parent directory.

```python
- seeing-machines/
  - SM02-ElieZananiri/
    - src/
    - addons.make
```

* **OPTIONAL** In true ITP fashion, you can make a blog post about your project. If you do, please send me the link!

* Post your project link to the `#assignments-25` channel on our Discord server. Do not send it by email. Do not send it as a DM.

  * Attach the packaged ZIP to your message.
  * If that does not work, upload it to Google Drive and send the link.
  * If you made a blog post or added your project to GitHub, send a link to that too.

Come to class with a working project on a working computer, and be prepared to talk and answer questions about it. Time allowing, some of you will demo your projects to the class!

Thank you!
