---
layout: post
title: Rust + WASM
---

I've been curious about the UI framework engineering opportunities of a minimal UI engine built in Rust and compiled into WASM. I'm going to document my journey here!

# Getting Started

Using the [Rust and WebAssembly](https://rustwasm.github.io/docs/book/) book

1. Install `rust-wasm`: `curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh`
2. Install `cargo-generate`: `cargo install cargo-generate`
3. Bootstrap an app: `cargo generate --git https://github.com/rustwasm/wasm-pack-template`
4. Build the bootstrapped app: `rust-wasm build`
5. Bootstrap a web app: `npm init wasm-app www` (see [create-wasm-app](https://github.com/rustwasm/create-wasm-app))
6. Install dependencies (from www directory): `npm install`
7. Link WASM module: `npm link ../pkg`
8: Update index.js: `import * as wasm from "wasm-game-of-life";`
9. Start the dev server: `npm start`

Question: How do I add defaults to function parameters as you would in ES6: `function greet(name = 'User') {}`

Answer: Using `Option`: http://xion.io/post/code/rust-optional-args.html + https://github.com/rustwasm/wasm-bindgen/pull/507

```rs
    let name = match name {
        None => String::from("User"),
        Some(x) => x
    };
```

Even better (via https://8thlight.com/blog/uku-taht/2015/04/29/using-the-option-type-effectively.html) ...

```rs
let name = name.unwrap_or(String::from("User"));
```