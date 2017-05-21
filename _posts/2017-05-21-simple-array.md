---
layout: post
title: Simple Array
tags: [data-structs]
---

In the simple array representation, chunks are represented by arrays of blocks. Unless you need to keep track of per-block state, this is almost certainly not the way to go. Let's understand why.

### Memory Usage
The simple array representation doesn't spare memory. Every block has a place for it to store its data, which let's say is 4 bytes. If you've got 10,000,000 blocks loaded, this adds up to 40MB, which can cause a little delay while reading the blocks from the disk while loading a world and can cause a little more delay when downloading the world from a multiplayer server.

If in your game players typically don't interact with more blocks (e.g. spend lots of time in one place), then this isn't such a problem for you, but if players can move around quickly and need to be constantly loading more chunks, they're probably going to notice the delays with this representation.

### Graphics Performance
todo
