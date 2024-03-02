---
title: "Bevy Map Gen Pt1"
date: 2023-05-11T18:36:09-05:00
---

For simplicity, the map generator will generate maps on a 2D square grid. I'm using the game engine [Bevy](https://bevyengine.org/), for those unfamiliar which uses an ECS (Entity Component System). Plenty of reading on the internet if you need more info on what those are!

One of the first implementation decisions, is how do I store the tiles within the grid. Is each tile an entity? Or do I have a "grid" or "tilemap" entity, that contains an array of  tiles?

In keeping with the *spirit* of an ECS, I've decided to make each tile its own entity. That makes more sense in my brain for now, although we'll see how that goes. It's possible we'll need to change that up in the future for implementation for performance reasons.

In addition, I am aware that there are prebuilt tilemap solutions for bevy, but for now I'm gonna try to roll my own. Again, that may change in the future.

## Let's get into some code

I've started by defining a very simple ```Tile``` component:

```rust
#[derive(Component)]
struct Tile {
    elevation: i32,
}

```

While our map will be 2D, so we won't actually haveelevation, I'll be using ```elevation``` as a value to influence and generate the map itself.


Now that we have a tile component, we need to start generating a grid of tile entities.
