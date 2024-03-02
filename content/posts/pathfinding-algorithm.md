---
title: "Pathfinding in Rust - Part 2 - BFS"
date: 2024-01-18T18:05:19-06:00
draft: false
---

In the last section, we created a grid-based map with tiles and code to render it. That puts us in a good position to start the pathfinding logic.

### Start and end

We'll need a start and end point to track where we're pathfinding to and from. Later on, I'd like to set this via mouse input, but for now, let's just make them hard coded variables:


```rust
    let start = Position { x: 0, y: 0 };
    let end = Position { x: 7, y: 7 };
```

The basic idea of a pathfinding algorithm is to iterate through the grid by checking the neighbors of any one tile. So let's implement a neighbors function on our Map class. First, some convenience functions to generate vectors matching our coordinate system:

```rust
    pub fn up() -> Self {
        Self { x: 0, y: -1 }
    }
    pub fn down() -> Self {
        Self { x: 0, y: 1 }
    }
    pub fn left() -> Self {
        Self { x: -1, y: 0 }
    }
    pub fn right() -> Self {
        Self { x: 1, y: 0 }
    }
```

And overloading the addition operator for the Position struct will be helpful as well:
```rust
impl ops::Add<Position> for Position {
    type Output = Position;
    fn add(self, _rhs: Position) -> Position {
        Position {
            x: self.x + _rhs.x,
            y: self.y + _rhs.y,
        }
    }
}

```

Now, the neighbors function becomes easy:
```rust
    fn neighbors(&self, position: &Position) -> Vec<Position> {
        let above = position + Position::down();
        let below = position + Position::down();
        let left = position + Position::left();
        let right = position + Position::right();
        vec![above, below, left, right]
    }
```

With the neighbors function complete, we can start the actually algorithm. Because I want to be able to split the algorithm out into steps for visualization purposes, I've created a class so that state of the graph traversal can be stored.

```rust
pub struct Pathfinder<'a> {
    map: &'a Map,
    start: Position,
    end: Position,

    frontier: VecDeque<Position>,
    reached: HashSet<Position>,
    current: Position,
}

impl<'a> Pathfinder<'a> {
    pub fn new(map: &'a Map) -> Self {
        Self {
            map: map,
            start: Position::up(),
            end: Position::up(),
            current: Position::up(),
            frontier: VecDeque::new(),
            reached: HashSet::new(),
        }
    }

    pub fn bfs(&mut self, start: Position) {
        self.start = start.to_owned();
        self.frontier.push_back(start.to_owned());
        self.reached.insert(start.to_owned());

        while !self.frontier.is_empty() {
            self.current = self.frontier.pop_front().unwrap().to_owned();
            for next in self.map.neighbors(&self.current) {
                if !self.reached.contains(&next) {
                    self.frontier.push_back(next.to_owned());
                    self.reached.insert(next);
                }
            }
        }
    }
}
```

The basic bfs algorithm is fairly simple. Create a reached set to track which nodes we have visited, and a frontier to track the edge of the reached set. Then, keep iterating through the frontier, adding neighbors of the current frontier node to the frontier, and marking the current frontier as reached.

However, I run it and...

...we get a crash.

I looked back and realized the neighbors function isn't checking if a neighbor is out of bounds of the map. So I went back and added some bounds checking methods to the map class:

```rust
    fn index_safe(&self, x: i32, y: i32) -> Option<usize> {
        if x < 0 || x > self.width || y < 0 || y > self.height {
            return None;
        }
        Some(self.index(x, y))
    }
    fn index_safe_pos(&self, pos: Position) -> Option<usize> {
        self.index_safe(pos.x, pos.y)
    }
    fn in_map(&self, pos: &Position) -> bool {
        self.index_safe_pos(pos.to_owned()).is_some()
    }
    fn unindex(&self, index: usize) -> Position {
        let index_i32: i32 = index.try_into().unwrap();
        Position {
            x: (index_i32 % self.width).try_into().unwrap(),
            y: (index_i32 / self.width).try_into().unwrap(),
        }
    }
```
Perhaps not the cleanest, but I like small functions that don't get to complicated.

The neighbors function then becomes:
```rust

    pub fn neighbors(&self, position: &Position) -> Vec<Position> {
        let mut n = Vec::new();
        let above = position + Position::up();
        if self.in_map(&above) {
            n.push(above);
        }
        let below = position + Position::down();
        if self.in_map(&below) {
            n.push(below);
        }
        let left = position + Position::down();
        if self.in_map(&left) {
            n.push(left);
        }
        let right = position + Position::down();
        if self.in_map(&right) {
            n.push(right);
        }

        n
    }
```

And with that, the bfs appears to be working. In the next stage, we'll add some visualization!

![checkerboard](/checkerboard.png)


Looks like it works! Next time we should have everything we need to start pathfinding!