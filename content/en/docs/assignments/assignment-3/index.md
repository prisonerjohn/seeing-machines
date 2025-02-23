---
title: "Assignment 3"
description: ""
date: 2022-10-07T20:53:02-04:00
lastmod: 2022-10-07T20:53:02-04:00
draft: false
images: []
menu:
  docs:
    parent: "assignments"
    identifier: "assignment-3"
weight: 300
---

## Depth Effect

### Instructions

Your assignment is to create an original application using openFrameworks and a depth sensing camera. This will give you an opportunity to get familiar with combined depth and color images, and explore point clouds in 3D.

* You can use any depth sensing device you want. There should be plenty of options in the ER.
* Your application can be either 2D or 3D (or any-D), but should use depth data as input.
* You can use any of the computer vision (CV) algorithms and functions we have seen so far in this course. This is not required, but might be useful depending on what you are trying to do.
* If you have an idea but are not sure how to do it, ask about it on Discord and we can break it down and figure it out together.
* Expose your parameters and use a GUI or some form of controller to tweak them for best results.
* Some of you will demo your project in class. Your effect should therefore work in the conditions of our classroom (size, layout, light, etc.)

## Delivery

* Name your project `SM03-FirstLast` where **First** is your first name and **Last** is your last name.

```python
- OF/
  - apps/
    - seeing-machines/
      - SM03-ElieZananiri/
        - src/
        - bin/
        - addons.make
        - SM03-ElieZananiri.sln
        - SM03-ElieZananiri.vcxproj
        - ...
```

* Only submit the necessary files to rebuild your project.

  * This includes sources, the `addons.make` file, and any resources in your `data` folder.
  * No project or compiled files.
  * In the example above, you would only keep the `src` folder, `addons.make` file, and `bin/data` if you are using any external assets.
  * Zip the `SM03-ElieZananiri` parent directory.

```python
- seeing-machines/
  - SM03-ElieZananiri/
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
