---
layout: post
title: 'Rust + WASM: Getting Started'
tags: rust, wasm
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

## Default Function Parameters

**Question:** How do I add defaults to function parameters as you would in ES6: `function greet(name = 'User') {}`?

**Answer:** Using `Option`: [xion](http://xion.io/post/code/rust-optional-args.html) + [wasm-bindgen](https://github.com/rustwasm/wasm-bindgen/pull/507)

```rs
    let name = match name {
        None => String::from("User"),
        Some(x) => x
    };
```

Even better (via [8thlight](https://8thlight.com/blog/uku-taht/2015/04/29/using-the-option-type-effectively.html)) ...

```rs
let name = name.unwrap_or(String::from("User"));
```

# Implement Game of Life

Copy and Paste a bunch of code to [build the Game of Life](https://rustwasm.github.io/docs/book/game-of-life/implementing.html).

* Question: Can I use the [`rand`](https://crates.io/crates/rand) crate instead of [`js-sys`](https://crates.io/crates/js-sys)?
* Answer:
Adding it will compile but bombs on the web with the folllowing error:
```
RuntimeError: "unreachable executed"
```
Enabling the `wasm-bindgen` feature for the `rand` crate fixes that up!
```
[dependencies.rand]
version = "0.7"
features = ["wasm-bindgen"]
```

## Adding configurability

I was curious to add the ability to pass in the size and eventually a seed array for the game. My went a little too big for my first attempt and tried to use an optional struct and then default out both the struct and its members.

```rs
#[wasm_bindgen]
pub struct UniverseOptions {
    width: u32,
    height: u32
}
#[wasm_bindgen]
impl Universe {
    pub fn new(opts: Option<UniverseOptions>) -> Universe {
        // let UniverseOptions {width, height} = opts;
        let UniverseOptions {width: width_opt, height: height_opt} = opts.unwrap_or(UniverseOptions {
            width: None,
            height: None
        });
        let width = width_opt.unwrap_or(64);
        let height = height_opt.unwrap_or(64);
        ...
    }
}
```

```js
const universe = Universe.new({width: 20, height: 20});
```

That compiled and ran but I was always seeing the default values so I reverted to a much simpler approach to validate it was possible.

```rs
pub fn new(width: u32, height: u32) -> Universe {
    ...
}
```

That worked fine so I started moving back towards my original goal. I tried to debug the execution in Firefox v68 and could step through the WASM code but I can't really read it except to see where it was defaulting to 64:

```wasm
i32.const 64
```

I guessed that Rust and WASM was unable (at least with my current implementation) to convert my generic object to a `UniverseOptions` so I added a `new()` factory for it and (after adding `#[wasm_bindgen]` to the `impl`) invoked that from JS and had a working version! <span alt="tada icon">🎉</span>

```rs
#[wasm_bindgen]
pub struct UniverseOptions {
    width: u32,
    height: u32
}

#[wasm_bindgen]
impl UniverseOptions {
    pub fn new(width: u32, height: u32) -> UniverseOptions {
        UniverseOptions {
            width,
            height
        }
    }
}

#[wasm_bindgen]
impl Universe {
    pub fn new(opts: UniverseOptions) -> Universe {
        let UniverseOptions {width, height} = opts;
        ...
    }
}
```

```js
import { Universe, UniverseOptions } from "wasm-game-of-life";
const universe = Universe.new(UniverseOptions.new(32, 32));
```

This finally got me back to my original implementation which supports:
* Omitting `UniverseOptions`: `Universe.new()`
* Omitting `height`: `Universe.new(UniverseOptions.new(32))`
* Omitting `width`: `Universe.new(UniverseOptions.new(null, 32))`

```rs
#[wasm_bindgen]
pub struct UniverseOptions {
    width: Option<u32>,
    height: Option<u32>
}

#[wasm_bindgen]
impl UniverseOptions {
    pub fn new(width: Option<u32>, height: Option<u32>) -> UniverseOptions {
        UniverseOptions {
            width,
            height
        }
    }
}

#[wasm_bindgen]
impl Universe {
    pub fn new(opts: Option<UniverseOptions>) -> Universe {
        let UniverseOptions {width: width_opt, height: height_opt} = opts.unwrap_or(UniverseOptions {
            width: None,
            height: None
        });
        let width = width_opt.unwrap_or(64);
        let height = height_opt.unwrap_or(64);
        ...
    }
}
```

## Bonus: Width or Size

One final exploration for today: when width is provided and height isn't, use width for both. This turned out to be pretty easy.

```rs
#[wasm_bindgen]
impl UniverseOptions {
    pub fn new(width: Option<u32>, height: Option<u32>) -> UniverseOptions {
        UniverseOptions {
            width,
            height: match height {
                None => width,
                Some(h) => Some(h)
            }
        }
    }
}
```

> I'm guessing there's a more efficient way to write that `match` but I haven't learned it so this'll do for now!
