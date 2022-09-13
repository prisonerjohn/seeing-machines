---
title: "Foreword"
description: ""
lead: ""
date: 2022-08-16T18:37:52-04:00
lastmod: 2022-08-16T18:37:52-04:00
draft: false
images: []
menu:
  docs:
    parent: "prologue"
    identifier: "foreword"
weight: 110
toc: true
---

## Introductions

A bit about me:

* [Beta Movement](https://betamovement.net/)

A bit about you:

* What did you do before ITP?
* Tell me about your programming experience.
* What are you hoping to get out of the class?

## Senses

### What is a sense?

A capacity that allows organisms to perceive the conditions or properties of things, either around them or internally.

### Human senses

We have traditionally only considered five human senses:

* Sight
* Hearing
* Smell
* Taste
* Touch

{{< details "Which of these would you say we use more predominantly?" >}}

Neurologist Dr. Wilder Penfield conceived the [Sensory Homuncilus](https://en.wikipedia.org/wiki/Cortical_homunculus), a physical representation of how the human body would look if the various body parts were sized in proportion to the cortical area used for their specific sensory functions.

{{< image src="https://upload.wikimedia.org/wikipedia/commons/c/c4/1421_Sensory_Homunculus.jpg" alt="A 2-D cortical sensory homunculus" width="360px" align="center" >}}

{{< image src="https://www.sharonpricejames.com/uploads/1/1/2/8/112878735/banner-image_1_orig.jpg" alt="3-D interpretation by Sharon Price James" width="360px" align="center" >}}

This is a simplification, but demonstrates that touch is the most predominant sense, followed by taste, hearing, smell, and finally sight.

{{< /details >}}

We also have many other senses, which we use in our daily life but are less obvious:

* Equilibrium
* Temperature
* Pain
* Thirst and hunger
* Direction
* Time
* Etc.

### Modeling machines

In order to get machines to understand their environment, we tend to outfit them with sensors that are similar to our own senses.

{{< details "What are some sensors that we use on computers?" >}}

* Sight
  * Digital camera
  * IR receiver
* Hearing
  * Microphone
* Touch
  * Trackpad
  * Pressure sensor
  * Keyboard
* Equilibrium
  * Gyroscope
* Direction
  * Magnetometer
  * Compass

{{< /details >}}

You've probably used some of these in your previous classes and projects.

## The right tool for the job

The focus of Seeing Machines will be to use sensors with computers (rather than microcontrollers), for the purpose of building successful interactive experiences.

The devices we will use will have SDKs (software development kits) and interfaces for many platforms and languages. This is great as it allows us to use something we are already familiar with, however some tools are better suited than others for specific tasks. For example, Python is great at text and language processing, Max is best at sound analysis, and Unity is ideal to get up and running with VR.

A lot of these platforms use very similar paradigms, and the difficulty of moving from one to the other tends to be more about getting familiar with a new environment and different coding syntax than anything else.

The majority of the programming for this class will be done in [openFrameworks](https://openframeworks.cc/) (OF) and we will sometimes detour to another platform when it makes sense. While C++ can be daunting, it is a very high performance language that is widely used, and OF takes a lot of the initial hurdles away!

About halfway through the semester, we will have a lecture on communication, where we will learn various methods for different pieces of software and hardware "talk" to each other.
