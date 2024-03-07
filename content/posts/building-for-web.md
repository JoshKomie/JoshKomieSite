---
title: "Building rust for web with WASM"
date: 2024-01-25:05:19-06:00
draft: false
---

One of the goals for this project is to produce runnable example in the web browser using web assembly. So let's dive in to that.

### Building

Luckily, our graphics library macroquad has comes with WASM build support. The documentation lists the following commands to build for WASM:

```
rustup target add wasm32-unknown-unknown
cargo build --target wasm32-unknown-unknown
```

Running them goes smoothly and produces a .wasm file. I've copied that over into the project for this blog, inserted a bit of html, and:

<canvas id="glcanvas" tabindex='1' width=400 height=400></canvas>
<!-- Minified and statically hosted version of https://github.com/not-fl3/macroquad/blob/master/js/mq_js_bundle.js -->
<script src="https://not-fl3.github.io/miniquad-samples/mq_js_bundle.js"></script>
<script>load("/pathfinder.wasm");</script>

Voila!!! We can now put running examples straight into the blog!