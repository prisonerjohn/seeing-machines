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
* Set black `0` as the pixel value for a dead cell, and white `255` as the pixel value for a live cell.
* Set the initial values to whatever you want. You can use random values or try a pattern.
* Neighbors are the pixels on the top-left, top-center, top-right, middle-left, middle-right, bottom-left, bottom-center, and bottom-right (8 neighbors total).
* Make sure to handle edge cases appropriately! You can either ignore the invalid neighbors or wrap around the texture.

We will follow the same rules as the original game of life:

1. Isolation: a live ⬜ cell with less than `2` live neighbors will die 😥.
1. Overcrowding: a live ⬜ cell with `4` or more neighbors will die 😵.
1. Reproduction: a dead ⬛ cell with exactly `3` live neighbors will live 🐣.

Here is some pseudo-code representing these rules:

```python
for (each cell in image):
    count live neighbors
    if (cell is live):
        if (num live neighbors < 2):
            cell dies
        if (num live neighbors > 3):
            cell dies
    if (cell is dead):
        if (num live neighbors == 3):
            cell lives
```

{{< video ratio="1x1" attributes="controls autoplay loop" mp4-src="game-of-life.mp4" >}}

{{< details "Hint: How can we fill an <code>ofImage</code> with values?" >}}

* [`ofImage.allocate()`](https://openframeworks.cc/documentation/graphics/ofImage/#show_allocate) will allocate memory for the image without having to load a file from disk.
* [`ofImage.getPixels()`](https://openframeworks.cc/documentation/graphics/ofImage/#show_getPixels) will return an `ofPixels` object which we can use to access the pixel data.
* `ofImage.getPixels()` makes a copy of the pixel data. After we make our edits, we need to save the new data back to the `ofImage` using [`ofImage.setFromPixels()`](https://openframeworks.cc/documentation/graphics/ofImage/#show_setFromPixels).

```cpp
lifeImg.allocate(40, 30, OF_IMAGE_GRAYSCALE);

ofPixels lifePix = lifeImg.getPixels();
for (int y = 0; y < lifeImg.getHeight(); y++)
{
  for (int x = 0; x < lifeImg.getWidth(); x++)
  {
    if (ofRandomuf() < 0.5)
    {
      lifePix.setColor(x, y, ofColor(0));
    }
    else
    {
      lifePix.setColor(x, y, ofColor(255));
    }
  }
}
// getPixels() makes a copy of the pixels, so we need to 
// use setFromPixels to set the new values back on the image.
lifeImg.setFromPixels(lifePix);
```

{{< /details >}}

### Bonus Points!

If you are looking for an additional challenge, try adding the following extra features:

1. Press a key to pause / start the simulation.
1. Press a key to reset the grid to random values.
1. Press a key to set the grid to all live cells, and another for all dead cells.
1. Click the mouse on a cell and toggle its state!

## Delivery

* Name your project `SM01-FirstLast` where **First** is your first name and **Last** is your last name.

```python
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

```python
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

<!-- ## Solution

Here are example projects for a [basic solution](sm01-ElieZananiri-basic.zip) and a [fancy solution](sm01-ElieZananiri-fancy.zip) (with all the bonus features).

A few things to watch out for:

* While iterating through the pixels, we do not want to read values from the same array we are writing to. If we do this, we will be reading values that we are modifying and will get unexpected results.
* When counting neighbors, we want to make sure to skip the current pixel. We need to look at the 8 surrounding pixels only.

-->
