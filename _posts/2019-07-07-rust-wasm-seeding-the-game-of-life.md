---
layout: post
title: 'Rust + WASM: Seeding the Game of Life'
---

After getting a [basic game of life running in Rust + WASM]({{ site.baseurl }}{%
post_url 2019-07-04-rust-wasm-getting-started %}), I wanted to expand it a bit
further to allow seeding the initial state. Initially, I planned to allow an
array of flags to seed any value but decided instead to use strings instead. It
proved to be a bit more difficult but also more interesting.

<!--more-->

# Adding `seed` arg

I started by adding `seed` to my existing `UniverseOptions` struct as
`Option<String>`. Omitting the seed will cause the app to fall to back to a
random seed as before.

```rs
impl UniverseOptions {
    pub fn new(width: Option<u32>, height: Option<u32>, seed: Option<String>) -> UniverseOptions {
        UniverseOptions {
            width,
            height: match height {
                None => width,
                Some(h) => Some(h)
            },
            seed
        }
    }
}
```

In `Universe`, I extract the value and default it to an empty string.

```rs
impl Universe {
    pub fn new(opts: Option<UniverseOptions>) -> Universe {
        let UniverseOptions {width: width_opt, height: height_opt, seed: seed_opt} = opts.unwrap_or(UniverseOptions {
            width: None,
            height: None,
            seed: None
        });
        let width = width_opt.unwrap_or(64);
        let height = height_opt.unwrap_or(64);
        let seed = seed_opt.unwrap_or(String::from(""));
        ...
    }
    ...
}
```

I find a struct to be good way to handle optional args from JS but I'd like a
better bridge than using `UnverseOptions.new()` that would allow me to pass a
generic object and it be cast into correct struct. I'm guessing such a thing
exists already but I just haven't discovered it yet.

# Refactoring Out Population

The code in `Universe::new` was quickly getting out of hand so my first step was
to refactor the population logic into a new `fill_cells` method. Initially, it
was pretty redundant. I didn't save the code but something like:

```rs
fn fill_cells(width: u32, height: u32, seed: &str) -> Vec<Cell> {
    if seed.len() == 0 {
        // generate random cells and return
    } else {
        // generate cells from text and return
        // this code never worked at this point :/
    }
}
```

By the end, I eventually moved to the following which only generated the cells
once for either branch and delegated populating specific cells to one of two
methods.

```rs
fn fill_cells(width: u32, height: u32, seed: &str) -> Vec<Cell> {
    let mut cells: Vec<Cell> = (0..width * height).map(|_| Cell::Dead).collect();

    if seed.len() == 0 {
        fill_random(&mut cells);
    } else {
        fill_seed(&mut cells, height, width, seed);
    }

    cells
}
```

# Defining a Font

Since I'm accepting strings, I need a way to convert a given character into a
bitmapped glyph. It took me some searching but eventually landed on a constant
`u8` array with each character being represented by 20 elements of the array.
`1` represents an alive cell and `0` a dead cell. This approach allowed me to
"see" the character in code so I could predict how it would render on the web.
It was a bit laborious to create them all but likely took less time than
searching for either an algorithm to create them or an example to steal. 

> As a future exercise, I've considered converting these to bit flags which
> could be stored more efficiently than an array of `u8`.

```rs
const FONT: [u8; 20 * 26] = [
    // A
    0, 1, 1, 0,
    1, 0, 0, 1,
    1, 1, 1, 1,
    1, 0, 0, 1,
    1, 0, 0, 1,
    // B
    1, 1, 1, 0,
    1, 0, 0, 1,
    1, 1, 1, 0,
    1, 0, 0, 1,
    1, 1, 1, 0,
    // ... etc ...
]
```

This matches the design of the `cells` array and requires similar math to
convert from the local coordinate system of the font to that of the `cells`
array. Below is an early working version.

I'm tracking the position of the next character with `col_offset`. Each
character is 4 bits wide by 5 bits tall and I'm adding 1 bit of letter spacing
so each character printed will increase the `col_offset` by 5. I iterate over
the characters, convert to the index in my font (`65` is the character code for
`A`), and map the glyph bits onto the corresponding cells.

```rs
let mut col_offset = 0;
for c in seed.chars() {
    let index = c as i32 - 65;

    // guard against overflowing
    if col_offset >= width - 4 {
        break;
    }

    if index >= 0 && index < FONT.len() as i32 {
        for i in 0..20 {
            let cell_index = ((i - i % 4) / 4) * width + i % 4 + col_offset;
            cells[cell_index as usize] = match FONT[(index * 20 + i as i32) as usize] {
                1 => Cell::Alive,
                _ => Cell::Dead
            };
        }

        col_offset += 5;
    }
}
```

# Unit Test!

Unit testing is so important in nearly all projects to deliver a quality
product. In projects with multiple layers, particularly when there is
translation between those layers, unit testing is critical. I would frequently
hit "unreachable executed" errors when something failed in WASM-land and
debugging was difficult. Validating that the code worked correctly in Rust helps
to limit interop debugging to interop concerns.

TL; DR: Write unit tests!

# Adding Centering

I initially avoided text positioning by calculating the "canvas" size in JS
based on the text size. So for example, the text "DAD" would be generated inside
a 15x5 canvas. This produced some interesting results on its own since the text
ran the the bounds and the game of life algorithm wraps around the bounds.

![game of life with DAD as seed string]({{ site.baseurl }}/images/dad.gif)

Always up for a challenge, I decided to add support to center the text
vertically and horizontally in the canvas which turned out to be pretty
straightforward given my current implementation. The existing `col_offset`
variable could be repurposed to start at a non-zero to adjust from where the
text began. Vertical centering required a new variable, `row_offset`, which was
calculated as a function of the `height` of the canvas.

```rs
let row_offset = ((height - 5) / 2) as usize;
```

## Rounding Down

In this non-subpixel, anti-aliasing free world, the calculations need to land on
whole numbers. As a regular JS engineer, my first instinct for this calculation
was to round the value down.

```js
var rowOffset = Math.floor((height - 5) / 2);
```

This would be possible in Rust too (I'm assuming: I didn't explicitly try) but
casting (or whatever Rust calls it) the floating point value into `usize` did
the trick here.

## Type compatibility

I'm still coming to grips with working in a strongly typed language again. I've
worked in C, C++, and Java in my past but am rusty (pun intended). I've
discovered `as` to coerce types around but find I'm still pretty inefficient. I
also expect types (e.g. `u32` and `usize` to be usable together) though they
can't. I understand why but have been "spoiled" by JS for too long I guess.

# Extending character support

The last feature I wanted to add for this iteration was some extended character
support. I'm not interested in creating a complete font but wanted to at least
support spaces and the '+' character (so I could output "Welcome to Rust +
WASM").

I could have added space as a glyph in my font but that seemed too large
visually so I decided to implement it uniquely as a 2 column width space. This
implementation choice affected two parts: width calculation for centering and
text rendering.

> In retrospect, I might have chosen to render the text into a separate buffer,
> measure it for centering, and then copy into the output buffer. But I didn't
> here. :)

For the width calculation, I refactored out a new function, `calc_seed_width`,
which accepts the seed and returns the width. I also added a `has_char` function
to help filter the text based on the supported glyphs in my font.

```rs
fn has_char (c: char) -> bool {
    c >= 'A' && c <= 'Z'
}

fn calc_seed_width (seed: &str) -> usize {
    let mut width: usize = 0;

    for c in seed.chars() {
        if c == ' ' {
            width += 2;
        } else if has_char(c) {
            width += 5;
        }
    }

    width
}
```

To support text rendering, I did some further refactoring to the font indexing
in order to support the "+" character which didn't line up to its character code
position. Using `Option` allowed me a more intuitive API than "less than zero"
for valid indices which I adopted in an updated `has_char` function.

```rs
fn get_font_index (c: char) -> Option<usize> {
    if c >= 'A' && c <= 'Z' {
        Some(c as usize - 65)
    } else if c == '+' {
        Some(27)
    } else {
        None
    }
}

fn has_char (c: char) -> bool {
    match get_font_index(c) {
        None => false,
        _ => true
    }
}
```

`fill_seed` before space and "+" support:

```rs
for c in seed.chars() {
    let index = c as i32 - 65;

    if index >= 0 && index < FONT.len() as i32 {
        ...
    }
}
```

... And after space and "+" support:

```rs
for c in seed.chars() {
    if c == ' ' {
        col_offset += 2;
    } else if let Some(index) = get_font_index(c) {
        ...
    }
}
```

# Updating the JavaScript side

In order to test more text combinations, I took the lazy approach of plucking
the value from the query string. Since the value of `document.location.search`
includes the leading "?", I stripped that using `.substring(1)`.

```js
const str = document.location.search.substring(1) || 'Rust';
```

However, I quickly discovered that didn't work right because the value was URL
encoded ... I should have remembered that but didn't.

```js
const str = decodeURI(document.location.search.substring(1)) || 'Rust';
```

# Easter Eggs

With all of that in place, I thought it'd be fun to turn it into an easter egg
in which the text would render initially but trigger the game once the mouse
hovered. I won't cover the details here but it made for a fun exercise. 

![game of life with Welcome to Rust+WASM as seed string]({{ site.baseurl }}/images/rust+wasm.gif)

```js
import { Universe, UniverseOptions } from "wasm-game-of-life";

const pre = document.getElementById("game-of-life-canvas");
let universe;

const init = () => {
    const str = decodeURI(document.location.search.substring(1)) || 'Rust';
    universe = Universe.new(UniverseOptions.new(128, 7, str.toUpperCase()));
}

const render = () => {
    pre.textContent = universe.render();
    universe.tick();
}

let raf = null;
const renderLoop = () => {
    render();
    raf = window.setTimeout(renderLoop, 64);
};

const play = () => raf === null && renderLoop();

const stop = () => {
    if (raf !== null) {
        window.clearTimeout(raf);
        raf = null;
    }
};

pre.addEventListener('mouseover', () => play());
pre.addEventListener('mouseout', () => stop());
pre.addEventListener('mousedown', () => {
    stop();
    init();
    render();
});
pre.addEventListener('mouseup', () => play());

init();
render();
```