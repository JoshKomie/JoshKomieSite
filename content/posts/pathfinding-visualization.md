---
title: "Pathfinding in Rust - Part 3 - Visualization"
date: 2024-01-25:05:19-06:00
draft: false
---

In the last section, we wrote up a very basic BFS algorithm. Now, it's time to add some visualization

### Basics

I know we'll need to break up the BFS algorithm into steps, so I do so now:

```rust
    pub fn start_bfs(&mut self, start: Position) {
        self.start = start.to_owned();
        self.frontier.push_back(start.to_owned());
        self.reached.insert(start.to_owned());
    }

    pub fn step_bfs(&mut self) {
        if self.frontier.is_empty() {
            println!("algo complete!");
        }
        self.current = self.frontier.pop_front().unwrap().to_owned();
        for next in self.map.neighbors(&self.current) {
            if !self.reached.contains(&next) {
                self.frontier.push_back(next.to_owned());
                self.reached.insert(next);
            }
        }
    }

```

Now, we need to add a way to step through the algorithm. This could be done a variety of ways, but I'm thinking a simple button will work well here. Luckily, it's very easy to add a button with macroquad, our graphics library. For now, I throw it straight into main, although I'll likely move it later. Now, our main loop looks like this:

```rust
    let map = Map::checkerboard(8, 8);
    let mut pathfinder = Pathfinder::new(&map);
    pathfinder.start_bfs(Position { x: 0, y: 0 });
    loop {
        input(&map);
        if root_ui().button(Vec2 { x: 400., y: 400. }, "Next") {
            pathfinder.step_bfs();
        }
        render(&map);
        next_frame().await;
    }
```

The final step will be to add some visualization, so we can see what the current, next tiles are, as well as which are in the frontier and reached sets.

Let's do this by creating a render method, similar to the map renderer, that can "render" the current state of the pathfinder.

First I created a few helper methods and struct:

```rust
#[derive(Clone)]
struct Rect {
    pub x: f32,
    pub y: f32,
    pub w: f32,
    pub h: f32,
}


fn draw_rect(r: Rect, color: Color) {
    draw_rectangle(r.x, r.y, r.w, r.h, color);
}
fn draw_rect_border(r: Rect, color: Color, thickness: f32) {
    draw_rectangle_lines(r.x, r.y, r.w, r.h, thickness, color);
}

fn pos_to_display_rect(position: Position) -> Rect {
    Rect {
        x: (position.x * PIXELS_PER_TILE) as f32,
        y: (position.y * PIXELS_PER_TILE) as f32,
        w: PIXELS_PER_TILE as f32,
        h: PIXELS_PER_TILE as f32,
    }
}
```

And finally the render method. This took some tweaking, but I eventually got it looking the way I liked it:
```rust
fn render_pathfinder(pathfinder: &Pathfinder) {
    let current = pos_to_display_rect(pathfinder.current.to_owned());
    let frontier: Vec<Rect> = pathfinder
        .frontier
        .to_owned()
        .into_iter()
        .map(|p| pos_to_display_rect(p))
        .collect();
    let reached: Vec<Rect> = pathfinder
        .reached
        .to_owned()
        .into_iter()
        .map(|p| pos_to_display_rect(p))
        .collect();
    for f in reached {
        draw_rect(f.clone(), BLUE);
        draw_rect_border(f.clone(), GRAY, 1.0);
    }
    for f in frontier {
        draw_rect(f.clone(), GREEN);
        draw_rect_border(f.clone(), GRAY, 1.0);
    }
    draw_rect_border(current, YELLOW, 3.0);
}
```

And voila! We now have a nice visualization of our algorithm!