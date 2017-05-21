---
layout: post
title: Simple Array
tags: [data-structs]
---

In the simple array representation, chunks are represented by arrays of blocks. You'll need this representation if you need to keep track of per-block state, otherwise, this is almost certainly not the way to go.

### Memory Usage
The simple array representation doesn't spare memory. Every block has a place for it to store its data, which let's say is 4 bytes. If you've got 10,000,000 blocks loaded, this adds up to 40MB, which can cause a little delay while reading the blocks from the disk while loading a world and can cause a little more delay when downloading the world from a multiplayer server.

If in your game players typically don't interact with lots of blocks (e.g. they spend lots of time in one place), then this isn't such a problem for you, but if players can move around quickly and need to be constantly loading more chunks, they're probably going to notice the delays with this representation. Definitely don't intend on giving your players vast views of the the sprawling countryside.

If it's any consolation, the simple array representation is stable; the data in the representation is always the same for the same arrangement of blocks in a chunk. In addition, it always uses the same amount of memory regardless of the arrangement of blocks.

### Graphics Performance
If you haven't already, you'll probably want to read my [graphics post](https://lattice3f.github.io/blog/2017/05/21/graphics/) before reading this section.

The simple array representation gives us a few options when building our mesh. Let's go through them and their implications.

#### Unculled and Unmerged
Let's take every block and add a quad for each of its six faces. You can do the math on how many faces you'll end up with; if you want to have more than 1,000,000 blocks loaded, you're going to have a bad time with this one.

#### Culled
Let's take every block and add a quad for each of its visible faces. This has obvious benefits as explained in the graphics post. This is the approach that Minecraft takes. You can tell they use culling by putting your head inside a block (e.g. by placing two blocks of sand above you so it stacks over your head; creative mode recommended) and having a look around - you'll see that you can see through the blocks adjacent to the sand your head is in, which happens because those faces aren't included in the mesh because Minecraft decided you wouldn't ever be able to see them.

#### Culled and Merged
How about after culling the invisible faces, we merge faces together by finding rectangle covers for both sides of each slice. We can get decent graphics speed-ups, especially for rectangle-y structures like player's houses. The only trouble is actually generating these rectangle covers; 40MB is a fair bit of data to go through no matter how you do it.

We can start with the algorithm laid out in a great article at [0fps.net](https://0fps.net/2012/06/30/meshing-in-a-minecraft-game/), which I recommend reading if you're into statements backed by hard numbers and proofs. I'll explain the algorithm in brief here. Let's say we want to generate a rectangle cover for a slice for a particular material `m`:
```
- m m m
m m m m
m m m -
```
Let's call the horizontal axis the u-axis and the vertical axis the v-axis, since the actual axes depend on which way you've taken the slice. We start by adding a rectangle that covers only one of the cells with the smallest u-value, and among those cells the one with the smallest v-value:
```
- m m m
m m m m
1 m m -
```
Then we stretch that rectangle, first as far as we can in the u-direction without covering something we weren't supposed to cover:
```
- m m m
m m m m
1 1 1 -
```
Then we stretch it in the v-direction as far as we can:
```
- m m m
1 1 1 m
1 1 1 -
```
Then we make the next rectangle the same way, and continue until we're all covered. The next rectangle starts like this:
```
- 2 m m
1 1 1 m
1 1 1 -
```
And stretches like this (it can't stretch at all in the v-direction):
```
- 2 2 2
1 1 1 m
1 1 1 -
```
And then we add our final rectangle, which can't stretch at all:
```
- 2 2 2
1 1 1 3
1 1 1 -
```
In this case the cover had an optimal rectangle count of 3, but in general it's only guaranteed to be within a factor of 8 of the optimal (according to the proof at [0fps.net](https://0fps.net/2012/06/30/meshing-in-a-minecraft-game/); I'd be lying if I said I fully understood it).

My implementation of the algorithm can run on 8x8x8 block chunk in a single frame without the player even noticing, but you're like me and want to be able to multiple edits on multiple chunks in the same frame and have them all optimized by the next frame, this algorithm is going to give you some CPU trouble as it stands. If you're committed to making this work, though, there are lots of things you can do:

##### 1. Don't re-run the algorithm every time the chunk is edited
Let all the edits happen and run the algorithm each frame. This is kind of just common sense and speeds things up when the same chunk receives multiple edits in one frame.

##### 2. Don't re-run the algoithm every frame
When the chunk is edited during a frame, generate a non-optimized mesh (culled only, perhaps). Then, after it hasn't been edited for a while, generate an optimized mesh to replace the non-optimized one. This speeds things up when chunk edits tend to happen within the same few seconds (or whatever your optimization time interval is) and, as long as only a few chunks are in a non-optimized state at a time, shouldn't noticably impact graphics performance.

When I implemented this, for added stability, I would run the pending chunk optimizations only until the current frame had taken 30ms. After simultaneously editing a bunch of chunks, there would be a 5s delay before they queued themselves for reoptimization, after which up to 30ms of each frame would be spent optimizing them. The result was reasonably stable graphics but 100% CPU usage for a few frames starting 5s after the edit.

##### 3. Don't re-run the algorithm in full
One can imagine a number of acceleration structures for doing edits to the rectangle cover without recomputing it in its entirety - one example would be storing the results for each slice independently and, when a one-block edit is made, recomputing the covers only for the 3 slices that contained that block. I moved on from the simple array representation before trying this, but if you give it a shot, let me know how it works out!
