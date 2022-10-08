---
title : "Seeing Machines"
description: "A programming course where we'll explore various techniques and solutions for tracking and sensing people or objects in space."
date: 2022-08-16T18:24:18-04:00
lastmod: 2022-08-16T18:24:18-04:00
draft: false
images: []
---

ITP Fall 2022<br/>
Mondays 6:00pm - 8:30pm<br/>
370 Jay St, Room 413

Professor: Elie Zananiri<br/>
Contact Email: [ez377@nyu.edu](mailto:ez377@nyu.edu)<br/>
Office Hours: [Mondays 5:00pm-6:00pm by appointment](https://calendar.google.com/calendar/selfsched?sstoken=UUFtdUlOM1BOTFpsfGRlZmF1bHR8OTA2NTRjNjM2OTA5YjU0MTRhMjdjYjczYzc0ZTAwMTM)<br/>
Discord Server: [https://discord.gg/NGMGbjK8w3](https://discord.gg/NGMGbjK8w3)

## Overview

A programming course where we'll explore various techniques and solutions for tracking and sensing people or objects in space. Students will get familiar with the terminology and algorithms behind many sensing topics such as computer vision, depth cameras, positional tracking, and coordinate mapping. As these subjects are explored, we will also dig into communication, and how this information can be transmitted from one tool to another, for example using OSC, Spout/Syphon, MIDI, DMX/ArtNet. The goal being to use the right tool for the job and not limit ourselves to a particular piece of software.

We will mainly be working in C++ using the [openFrameworks](https://openframeworks.cc/) toolkit. C++ can be intimidating at first, but it is a fast, widely used, general purpose language that is worth the effort and rewarding, especially for this topic. Students are not expected to have prior experience with C++ or openFrameworks, but are required to have some programming experience and enjoy coding. You should come into the course knowing what a `variable`, `function`, `class`, `array`, and `for loop` is 🤓.

The first classes will consist of theory and in-class exercises covering these techniques, and remaining classes will be dedicated to a special project, which should use a combination of what we've learned to create a new work. Students will work in small groups to build this special project, but we'll review proposals, milestones, and work in progress collectively on every class, encouraging discussion and collaboration.

This is the second time this class is taught, its original name is [Sensing Machines](https://prisonerjohn.com/sensing-machines). It is a work in progress, so please let me know if there is any topic you would like to cover!

## Objectives

* Gain understanding of the various ways a computer system can "sense" its environment.
* Dive into the technical details to implement sensing systems.
* Become familiar with the distinct features of the languages and tools available to the programmer.
* Learn to use communication protocols to share data between different tools.
* Build cool projects.

## Evaluation

You should be an active participant in the class.

* Show up on time and pay attention.
* Ask questions, show work, participate in discussions.

There will be 4 assignments for the first part of the semester.

* Assignments should be completed individually.
* The objective of these is to make sure you understand what is covered in the class and get your hands dirty.
* If you are familiar with the techniques explored, get out of your comfort zone!
* Assignments will have an open-ended component and creativity is encouraged.

The second part of the semester will be dedicated to a larger project.

* Projects should be created by teams of 2 students.
* The objective is to build a fully fleshed out interactive, which can take the shape of an artwork, a performance, an installation, etc.
* Be prepared to show progress every week, starting with a proposal, progress milestones every week, and a final presentation of a working project on the last class.

The evaluation breakdown is as follows:

|                          |      |
|--------------------------|-----:|
| On-time Participation    | 20%  |
| Assignments              | 40%  |
| Final Project            | 40%  |

Pass/Fail means that anything below 80% is a fail.

I will notify you ahead of time if you are at risk of failing, and you can reach out to me at any time if you are concerned about your standing in the class.

## Schedule

| Date   | Topic          | Assignment | Recordings |
|:-------|:---------------|:-----------|:-----------|
| Sep 12 | [Foreword]({{< relref "docs/class-0/foreword" >}})<br/>[Intro to openFrameworks]({{< relref "docs/class-0/intro-to-of" >}}) | | [AUDIO](https://stream.nyu.edu/media/Seeing%20Machines%20Sep%2012%202022%20%5BAUDIO%5D/1_r1qgdqh9)<br/>[VIDEO](https://stream.nyu.edu/media/Seeing%20Machines%20Sep%2012%202022%20%5BVIDEO%5D/1_8uypx4sb) |
| Sep 19 | [Variables and Arrays]({{< relref "docs/class-1/variables-and-arrays" >}})<br/>[Images and Video]({{< relref "docs/class-1/images-and-video" >}}) | [Assignment 1 OUT]({{< relref "docs/assignments/assignment-1" >}}) | [AUDIO](https://stream.nyu.edu/media/Seeing+Machines+Sep+19+2022+%5BAUDIO%5D/1_z1u7mcj2)<br/>[VIDEO](https://stream.nyu.edu/media/Seeing+Machines+Sep+19+2022+%5BVIDEO%5D/1_hv93189q) |
| Sep 26 | [Computer Vision]({{< relref "docs/class-2/computer-vision" >}}) | Assignment 1 DUE<br/>[Assignment 2 OUT]({{< relref "docs/assignments/assignment-2" >}}) | [AUDIO](https://stream.nyu.edu/media/Seeing+Machines+Sep+26+2022+%5BAUDIO%5D/1_mrdyaszc)<br/>[VIDEO](https://stream.nyu.edu/media/Seeing+Machines+Sep+26+2022+%5BVIDEO%5D/1_tbda3i1y) |
| Oct 3  | [Intro to OpenCV]({{< relref "docs/class-3/intro-to-opencv" >}})<br/>[Object Tracking]({{< relref "docs/class-3/object-tracking" >}}) | | [AUDIO](https://stream.nyu.edu/media/Seeing+Machines+Oct+3+2022+%5BAUDIO%5D/1_el6vp5jl)<br/>[VIDEO](https://stream.nyu.edu/media/Seeing+Machines+Oct+3+2022+%5BVIDEO%5D/1_otqjbku5) |
| Oct 11 (TUE) | [Logging]({{< relref "docs/class-4/logging" >}})<br />[Depth Sensing]({{< relref "docs/class-4/depth-sensing" >}}) | Assignment 2 DUE<br/>[Assignment 3 OUT]({{< relref "docs/assignments/assignment-3" >}}) | |
| Oct 17 | <div class="blink">NO CLASS</div> | | |
| Oct 24 | Depth Images |  | |
| Oct 31 | Depth in 3D | Assignment 3 DUE<br/>Assignment 4 OUT | |
| Nov 7  | Networking<br/>Frame Buffers<br/>Texture Sharing |  | |
| Nov 14 | Classes<br/>Sound | Assignment 4 DUE | |
| Nov 21 | Mobile Development | Final Project OUT | |
| Nov 28 | Project Proposals<br/>Topic TBD | Project Proposal DUE | |
| Dec 5  | Milestone Check-In<br/>Topic TBD | | |
| Dec 12 | Final Presentations | Final Project DUE | |

## Academic Integrity

Plagiarism is presenting someone else’s work as though it were your own. More specifically, plagiarism is to present as your own: A sequence of words quoted without quotation marks from another writer or a paraphrased passage from another writer’s work or facts, ideas or images composed by someone else.

The core of the educational experience at the Tisch School of the Arts is the creation of original academic and artistic work by students for the critical review of faculty members.  It is therefore of the utmost importance that students at all times provide their instructors with an accurate sense of their current abilities and knowledge in order to receive appropriate constructive criticism and advice.  Any attempt to evade that essential, transparent transaction between instructor and student through plagiarism or cheating is educationally self-defeating and a grave violation of Tisch School of the Arts community standards.  For all the details on plagiarism, please refer to page 10 of the Tisch School of the Arts, Policies and Procedures Handbook, which can be found online at: [http://students.tisch.nyu.edu/page/home.html](http://students.tisch.nyu.edu/page/home.html)

## Accessibility

Please feel free to make suggestions to your instructor about ways in which this class could become more accessible to you.  Academic accommodations are available for students with documented disabilities. Please contact the Moses Center for Students with Disabilities at 212 998-4980 for further information.

## Counseling and Wellness

Your health and safety are a priority at NYU. If you experience any health or mental health issues during this course, we encourage you to utilize the support services of the 24/7 NYU Wellness Exchange [212-443-9999](tel:212-443-9999). Also, all students who may require an academic accommodation due to a qualified disability, physical or mental, please register with the Moses Center [212-998-4980](tel:212-998-4980). Please let your instructor know if you need help connecting to these resources.

## Use of Electronic Devices

Laptops will be an essential part of the course and may be used in class during workshops and for taking notes in lecture. Laptops must be closed during class discussions and student presentations.  Phone use in class is strictly prohibited unless directly related to a presentation of your own work or if you are asked to do so as part of the curriculum.

## Title IX

Tisch School of the Arts to dedicated to providing its students with a learning environment that is rigorous, respectful, supportive and nurturing so that they can engage in the free exchange of ideas and commit themselves fully to the study of their discipline. To that end Tisch is committed to enforcing University policies prohibiting all forms of sexual misconduct as well as discrimination on the basis of sex and gender.  Detailed information regarding these policies and the resources that are available to students through the Title IX office can be found by using the following link: [Title IX at NYU](https://www.nyu.edu/about/policies-guidelines-compliance/equal-opportunity/title9.html).
