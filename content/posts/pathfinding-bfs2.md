---
title: "Pathfinding in Rust - Part 4 - BFS cont"
date: 2024-01-18T18:05:19-06:00
draft: false
---

In the last section, we implemented a basic version of a breath first traversal. While that is nice and useful, let's use it to find a path.

To start, we need to update the reached HashSet from last time, into a HashMap<Position, Option<Position>>. We'll use this hash map to track where the tile was reached from. We can use this hash map as a sort of breadcrumb system to generate a path to a destination.

The code to do so looks like this:

```rust
    let mut current = self.end.to_owned();
    let mut path = Vec::new();
    while current != self.start {
        path.push(c.to_owned());
        match &self.came_from[&c] {
            Some(p) => c = p.to_owned(),
            None => {}
        }
    }
```