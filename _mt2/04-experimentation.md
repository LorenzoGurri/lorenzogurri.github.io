---
layout: single
title: Music & Tech 2 Experimentation and Setup
toc: true
toc_sticky: true
toc_label: "Ideas"
toc_icon: "file-alt"
excerpt: "Lets get our hands dirty"
author: Lorenzo Gurri
---

In this post, I start creating the project and experiment with implementing a plugin in Rust.


## Initalizing the project

First thing's first, we need to create the project itself. We can do that with Rust's 
package manager: `cargo`.

```
cargo new --lib ria_lv2
```

This will create a library package for us called `ria_lv2`. Next, we need to change the type of library
and then add some dependencies.

### Changing the library type

In Rust, there are different types of libraries you can create: **rlib**, **dylib**, and **cdylib**.[^1]
**rlib**s are static Rust libraries that link to other Rust libraries. These are useful
for generating libraries that will stay within the Rust ecosystem. **dylib**s are 
silimar to **rlib**s but are dynamic instead of static. Similar to **dylib**s in terms of
its dynamicness, **cdylib**s serve the purpose of linking to C or C++ programs. For this project,
we want to create a **cdylib** to allow Rust to generate an interface that can plugin with the
DAW software.

We also want to add the lv2 library as a dependency, of course. Luckily, we can make all these changes
in one file, `Cargo.toml`.

```toml
[package]
name = "ria_lv2"
version = "0.1.0"
edition = "2021"

# rlib is the default library type,
# change it to cdylib
[lib]
crate-type = ["cdylib"]

# add the lv2 dependency and use the
# latest version
[dependencies]
lv2 = "0.6.0"
```

## Compile An Example Plugin

Now we need to actually compile a plugin. We can copy the example code from the `lv2`
crate's website[^2]:

```rust
// Import everything we need.
use lv2::prelude::*;

#[derive(PortCollection)]
struct Ports {
    gain: InputPort<Control>,
    input: InputPort<Audio>,
    output: OutputPort<Audio>,
}

#[uri("https://github.com/RustAudio/rust-lv2/tree/master/docs/amp")]
struct Amp;

impl Plugin for Amp {
    type Ports = Ports;

    type InitFeatures = ();
    type AudioFeatures = ();
    fn new(_plugin_info: &PluginInfo, _features: &mut ()) -> Option<Self> {
        Some(Self)
    }
    fn run(&mut self, ports: &mut Ports, _features: &mut (), _: u32) {
        let coef = if *(ports.gain) > -90.0 {
            10.0_f32.powf(*(ports.gain) * 0.05)
        } else {
            0.0
        };

        for (in_frame, out_frame) in Iterator::zip(ports.input.iter(), ports.output.iter_mut()) {
            *out_frame = in_frame * coef;
        }
    }
}
lv2_descriptors!(Amp);

```

This will generate the `libria_lv2.so` file in the `target/debug/` directory. We just compiled our
first plugin!

## Using an LV2 plugin

LV2 plugins are a little needy. Besides just the compiled library, we need two configuration files
to be able to import our plugin. One is called `manifest.ttl` and the other is the name of our library,
`libria_lv2.ttl`.[^3]

The `manifest.ttl` file is scanned by the host at start up to discover plugins. The `libria_lv2.ttl` file
is responsible for giving a full description of the plugin. We can copy these files from the samples 
offered by the [Rust LV2 Book](https://rustaudio.github.io/rust-lv2/chapter/amp.html)

Once we have these, we are ready to try out our LV2 plugn! We need to put each file in a folder
called `libria_lv2.lv2` and toss that folder into the `~/.lv2` folder. After we do that, we can
launch our DAWs and load our plugin! 

## Testing setup

For testing, I will be using Ardour and Reaper.

### Testing in Ardour

After moving the files over to the `~/.lv2` directory, Ardour will try to scan the plugin, but
runs into an error since our plugin is too primative and doesn't support inplace processing.

![Ardour Error](/assets/images/mt2_ardour_lv2_error.png)

After doing a little research, apparently the error is [solvable](https://discourse.ardour.org/t/trying-to-do-a-lv2-plugin-with-lv2-rust-but-lv2-inplacebroken-is-not-supported-by-ardour/106765/3)
and a PR on rust's `lv2` library has [fixed it](https://github.com/RustAudio/rust-lv2/pull/93).
All I need to do is follow the PR for how to use the new port types of `InPlaceInput` and `InPlaceOutput`.

### Testing in Reaper

After adding `~/.lv2` to the plug-in paths through Options > Preferences > Plug-ins > LV2, we can load in our plugin!

![Reaper LV2](/assets/images/mt2_reaper_lv2.png)

Reaper seems to accept the plugin with no problem and has a nice little Gain slider for us.

From here we have a nice testing setup and a place to start implementing!

[^1]: [Difference in library types](https://users.rust-lang.org/t/what-is-the-difference-between-dylib-and-cdylib/28847)
[^2]: [Sample Code](https://docs.rs/lv2/latest/lv2/)
[^3]: [Sample TTL files](https://rustaudio.github.io/rust-lv2/chapter/amp.html)
