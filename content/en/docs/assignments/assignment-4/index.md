---
title: "Assignment 4"
description: ""
lead: ""
date: 2022-10-27T19:18:46-04:00
lastmod: 2022-10-27T19:18:46-04:00
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "assignment-4"
weight: 400
toc: true
---

## Communication

### Instructions

Your assignment is to create two applications and use a communication protocol to exchange information between them.

* At least one of your applications needs to be written in openFrameworks. The other can be OF or anything else.
* You can build off of a previous assignment or class exercise. For example, you can send blobs from the [color tracker]({{< relref "docs/class-4/object-tracking" >}}) to a different application to control another object.
* Make sure to use an appropriate method of communication. Think of reliability, speed, data size, etc. when making your decision.
* If you have an idea but are not sure how to do it, ask about it on Discord and we can break it down and figure it out together.
* Expose your parameters and use a GUI or some form of controller to tweak them for best results.
* Some of you will demo your project in class. Your effect should therefore work in the conditions of our classroom (size, layout, light, etc.)

## Delivery

* Name your projects `SM04-FirstLast-XXX` where **First** is your first name and **Last** is your last name.

```python
- OF/
  - apps/
    - seeing-machines/
      - SM04-ElieZananiri-Sender/
        - src/
        - bin/
        - addons.make
        - SM04-ElieZananiri-Sender.sln
        - SM04-ElieZananiri-Sender.vcxproj
        - ...
      - SM04-ElieZananiri-Receiver/
        - src/
        - bin/
        - addons.make
        - SM04-ElieZananiri-Receiver.sln
        - SM04-ElieZananiri-Receiver.vcxproj
        - ...
```

* Only submit the necessary files to rebuild your project.

  * This includes sources, the `addons.make` file, and any resources in your `data` folder.
  * No project or compiled files.
  * In the example above, you would only keep the `src` folder, `addons.make` file, and `bin/data` if you are using any external assets.
  * Zip the `SM04-ElieZananiri-XXX` parent directories.

```python
- seeing-machines/
  - SM04-ElieZananiri-Sender/
    - src/
    - addons.make
  - SM04-ElieZananiri-Receiver/
    - src/
    - addons.make
```

* **OPTIONAL** In true ITP fashion, you can make a blog post about your project. If you do, please send me the link!

* Post your project link to the `#assignments-23` channel on our Discord server. Do not send it by email. Do not send it as a DM.

  * Attach the packaged ZIP to your message.
  * If that does not work, upload it to Google Drive and send the link.
  * If you made a blog post or added your project to GitHub, send a link to that too.

Come to class with a working project on a working computer, and be prepared to talk and answer questions about it. Time allowing, some of you will demo your projects to the class!

Thank you!
