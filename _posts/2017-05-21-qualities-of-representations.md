---
layout: post
title: Qualities of Representations
tags: [data-structs]
---

If you're making a voxel-based game, you'll need to pick a representation for your world. Representations have different qualities and failure modes; it's helpful to understand these and compare them to your needs to make the best decision.

Representations will typically feature chunks, which are collections of blocks that are rectangular pieces of the world e.g. a particular 8-by-8-by-8 volume.

### Memory Usage
Voxel games feature lots of blocks (often on the scale of 10,000,000 being loaded at any time), and if you're not clever this can eat up lots of RAM. In addition, if your game is multiplayer, memory-intensive representations will chew up bandwidth when sending chunks over the network.

### Graphics Performance
Rendering all 6 faces of all your 10,000,000 blocks is not very kind to your graphics card. A few techniques can be applied to reduce graphics strain dramatically, and some representations make these techniques much easier than others make them.

### CPU Usage
Sometimes the representation you're using isn't good for performing certain operations, and if your game is heavy in those operations, you're going to have a bad time.

### Bulk Updates
Some representations make it easy to do certain sets of operations all at once for virtually the cost of a single one of those operations. Each representation may have its own conditions under which a bulk update is easy, and if these conditions are likely in your game, it could be a compelling argument for using that representation.

### Stability
Some representations will always be the same for the current state of the blocks in a chunk. Some representations will depend on what series of operations led to the current state, and this can lead to various failure modes.

### Level-Of-Detail
Often in games, when things are far away, we don't care too much about the finer details of how they appear. Some representations make it easy to simplify chunks so that they look more or less the same from far away and ease our memory usage and/or graphics strain.
