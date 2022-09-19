---
title: "Assignment 1"
description: ""
date: 2022-09-19T08:43:20-04:00
lastmod: 2022-09-19T08:43:20-04:00
draft: false
images: []
menu:
  docs:
    parent: "assignments"
    identifier: "assignment-1"
weight: 100
---

## Conway's Game of Life

[Conway's Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life) is a cellular automata simulation where a 2D grid of cells evolve between alive and dead states.

* The Game of Life is a zero-player game, where you just set the initial state and the rules, and watch the evolution happen.
* A cell's state in the next frame depends on the state of its immediate neighbors in the current frame. It is set by counting the number of live or dead neighbors and applying the corresponding rule.
* Depending on this initial state and rules, interesting patterns can emerge where cells can appear to oscillate or travel across the board.

### Instructions

Your assignment is to build your version of the Game of Life using openFrameworks. This will give you an opportunity to play with an image's pixel data.

* Use an `ofImage` as your 2D canvas.
* Set `0` as the pixel value for a dead cell, and `255` as the pixel value for a live cell.
* Neighbors are the pixels on the top-left, top-center, top-right, middle-left, middle-right, bottom-left, bottom-center, and bottom-right.
* Make sure to handle edge cases appropriately! You can either ignore the invalid neighbors or wrap around the texture.

While Conway's version has its own specific rules, our version will use rules based on your NYU ID.

1. The first number in your ID is the number of dead neighbors required for a live cell to die.
1. The last number in your ID is the number of live neighbors required for a dead cell to come alive.
1. If the first and last number of your ID are the same, use the middle number instead for rule 2.

For example, my NYU ID is ez377. Here is some pseudo-code representing my rules:

```text
for (each cell in image):
    if (cell is dead):
        count live neighbors
        if (num live neighbors == 7):
            cell comes alive!
    else:
        count dead neighbors
        if (num dead neighbors == 3):
            cell dies :(
```

{{< video ratio="1x1" attributes="controls autoplay loop" mp4-src="game-of-life.mp4" >}}

## Delivery

* Name your project `SM01-FirstLast` where **First** is your first name and **Last** is your last name.

```text
- OF/
  - apps/
    - seeing-machines/
      - SM01-ElieZananiri/
        - src/
        - bin/
        - addons.make
        - SM01-ElieZananiri.sln
        - SM01-ElieZananiri.vcxproj
        - ...
```

* Only submit the necessary files to rebuild your project.

  * This includes sources, the `addons.make` file, and any resources in your `data` folder.
  * No project or compiled files.
  * In the example above, you would only keep the `src` folder, `addons.make` file, and `bin/data` if you are using any external assets.
  * Zip the `SM01-ElieZananiri` parent directory.

```text
    - seeing-machines/
      - SM01-ElieZananiri/
        - src/
        - addons.make
```

* **OPTIONAL** In true ITP fashion, you can make a blog post about your project. If you do, please send me the link!

* Post your project link to the `#assignments` channel on our Discord server. Do not send it by email. Do not send it as a DM.

  * Attach the packaged ZIP to your message.
  * If that does not work, upload it to Google Drive and send the link.
  * If you made a blog post or added your project to GitHub, send a link to that too.

Come to class with a working project on a working computer, and be prepared to talk and answer questions about it. Time allowing, some of you will demo your projects to the class!

Thank you!
