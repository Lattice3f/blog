---
layout: post
title: Graphics
tags: [data-structs]
---

In voxel-based games, the biggest graphics performance optimizations we can make come down to how many block faces we tell our graphics card to render. If we render fewer for the same number of blocks, then we can put more blocks on the screen at a time and/or turn up the graphics quality with higher resolution textures, better lighting qualities, etc.

### Background on Quads
In most game styles you just import your 3D models and the nitty-gritty of how the graphics works is handled for you by your game engine. In voxel-based games, our content is dynamic, so we'll need to be generating meshes for our world on-the-fly.

Graphics cards understand a mesh as a list of vertices (3 floats) and a list of triangles whose vertices are among those in the first list (3 ints; each int is the index of a vertex in the list of vertices). For example, the following lists represent a single triangle:

`Vertices: [(0.0, 0.0, 0.0), (0.0, 1.0, 0.0), (1.0, 0.0, 0.0)]`

`Triangles: [0, 1, 2]`

The order that we specify the vertices in is important; it determines the direction of the face. Faces are only rendered in one direction; if you look at a face from the other side it's invisible (which explains some of the graphical weirdness if you've every clipped through an object in a game).

To make a quad (quadrilateral), we just need two triangles. We could just list out six vertices and make the two triangles like so:

`Vertices: [(0.0, 0.0, 0.0), (0.0, 1.0, 0.0), (1.0, 0.0, 0.0), (1.0, 1.0, 0.0), (1.0, 0.0, 0.0), (0.0, 1.0, 0.0)]`

`Triangles: [0, 1, 2, 3, 4, 5]`

The graphics card will just take the ints from the triangles list 3 at a time and look up those vertices to draw the triangle. But you'll notice that two of the points are shared among both our triangles, and so to be clever we can reuse them like so:

`Vertices: [(0.0, 0.0, 0.0), (0.0, 1.0, 0.0), (1.0, 0.0, 0.0), (1.0, 1.0, 0.0)]`

`Triangles: [0, 1, 2, 3, 2, 1]`

If you go through the vertex lookups yourself you'll find that this produces the same 2 triangles.

### Face Culling
The first optimization we can make is to not draw faces that couldn't possibly be visible because they're occluded by other faces (e.g. if two blocks are adjacent to each other, the first block prevents anyone from seeing one of the faces on the second block and vice versa). This can give us order-of-magnitude decreases in the number of faces we need to render as it makes our number of faces proportional to the surface area of the world rather than the volume. For any normal voxel-based game, this is a must.

### Face Merging
Let's say we are looking at 4 blocks of the same material that are arranged in a square like this (I'll use `x` to indicate a block):
```
x x
x x
```
We don't need to draw a quad for the face of each block. We can draw one big quad that covers all of them and tile the texture so that players can't tell the difference.

What we want to avoid is four separate quads (I'll use numbers to partition the area into faces):
```
1 2
3 4
```
And instead merge the quads together when possible:
```
1 1
1 1
```
It isn't always clear how to best do this, but as long as we can get close enough, we can get huge improvements in our graphics performance. Note that because we'll no longer have all our vertices, using a vertex-lit shader is a no-go, so we'll have to use a pixel-lit shader. If you don't know the difference, I recommend a great website called [Google](https://www.google.com/).

Of course, this is just for one slice of a chunk, which is a set of blocks with some coordinate held at some fixed value (e.g. all blocks where y=6). To build the whole mesh, we need the faces for the tops and bottoms of all slices where z is fixed, both sides of all slices where x is fixed, and the fronts and backs of all slices where y is fixed.

When you have blocks with multiple materials, you send them to the graphics card as separate 'submeshes', one per material. This simplifies things for us quite a bit; we can consider each material independently and think about the problem in terms of which faces are on blocks of our current material and which ones are not. That is, let's say we have a slice of blocks made of air `a`, dirt `d`, and stone `s`:
```
a a a a
a d d d
d s s s
s s s s
```
We need to make meshes for each of these materials. Air we can ignore because it's invisible. The dirt part of our slice looks like this, where `-` indicates that we don't care about the face right now:
```
- - - -
- d d d
d - - -
- - - -
```
Then from this we build our dirt submesh, which we need two quads for:
```
- - - -
- 1 1 1
2 - - -
- - - -
```
Next we build our stone submesh; the stone part of our slice looks like this:
```
- - - -
- - - -
- s s s
s s s s
```
There are two ways to build the stone submesh with only two quads. The first:
```
- - - -
- - - -
- 1 1 1
2 2 2 2
```
And the second:
```
- - - -
- - - -
- 1 1 1
2 1 1 1
```
What we're doing here is generating a rectangle cover for the part of our slice, and sometimes finding good ones is expensive. In my posts on represenations of chunks, I'll explain some options for computing these from the representation, how some representations make getting a decent one fast, and what kind of failure modes these options have.
