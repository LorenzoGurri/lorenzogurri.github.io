---
layout: single
title: Music & Tech 2 Combining Filters
toc: true
toc_sticky: true
toc_label: "Combining Filters"
toc_icon: "file-alt"
excerpt: "Combining Filters"
author: Lorenzo Gurri
---

In this post, I talk about how to combine
filters statically as well as what the next steps
are for furthering the plugin.

## Filters in Series

So far, we have four filters implemented and can
run them in a mutually exclusive manner. This isn't what we want, however,
since the end goal of this plugin is to be able to dynamically
combine filters. To get started, we'll learn how to combine them statically 
(we set the filters to be enabled at compile time).

One way to do this is through creating a `Pipeline` struct that holds
all the effects we want enabled. Then, when we call the `run()` method,
the `in_sample` is passed through each effect until it is eventualy
returned as the `out_sample`.

However, Rust is a strongly typed language. This means that we can't
have a list of elements of different plugin types. For example, the below code
fails:


```rust
struct LFO;
struct LowPass;


fn main() {
    let mut v = Vec::new();
    v.push(LFO{});
    v.push(LowPass{});
}
```

```
error[E0308]: mismatched types
  --> src/main.rs:12:12
   |
12 |     v.push(LowPass{});
   |            ^^^^^^^^^ expected struct `LFO`, found struct `LowPass`

For more information about this error, try `rustc --explain E0308`.
error: could not compile `playground` due to previous error
```

## Traits

To get around this, we can utilize dynamic disbatch. This allows us to
use different types as long as they all implement the same `trait`.


Here is the above example but with dynamic disbatch:

```rust 
trait Filter {}

struct LFO;
impl Filter for LFO {}

struct LowPass;
impl Filter for LowPass {}

fn main() {

    let mut v: Vec<Box<dyn Filter>> = Vec::new();
    v.push(Box::new(LFO{}) as Box<dyn Filter>);
    v.push(Box::new(LowPass{}) as Box<dyn Filter>);
    println!("Dynamic!");
}
```

```
Dynamic!
```

It's important to note that using dynamic disbatch comes with
a small performance hit.

Traits are more powerful than just using them for dynamic disbatch.
Read more about them [here](https://doc.rust-lang.org/rust-by-example/trait.html).

## Effect Trait

**NOTE**: The code in this post is from commit [638191b](https://github.com/LorenzoGurri/ria_lv2/commit/638191bb7e878680d2b2060dbae86d0295d9f01a).
The code or file structure very well might have changed from then.

When we look at the [project](https://github.com/LorenzoGurri/ria_lv2)
on github, we can see the file structure is as follows

```
src/
 |- effects
      |
      |- allpass.rs
      |- amplfo.rs
      |- echo.rs
      |- highpass.rs
      |- lowpass.rs
      |- mod.rs
 |- lib.rs
 |- pipeline.rs
 |- ri.rs
```

When we take a look in the `mod.rs` file in the `effects` folder, we can see
the definition of a trait called `Effect`:

```rust
pub trait Effect {
    fn run(&mut self, in_sample: f32) -> f32;
}
```

This trait is implemented by all the effects and requires that they implement
the method `run(&mut self, in_sample: f32) -> f32`.

Taking a look at the `allpass.rs` file, we can see that the struct
`AllPass` has the trait `Effect` and therefore implements this `run()`
method.

```rust
pub struct AllPass;

impl Effect for AllPass {
    /// # Returns
    ///
    /// The input sample `in_sample`.
    fn run(&mut self, in_sample: f32) -> f32 {
        in_sample
    }
}
```

## Making a Pipeline

Now we can make a Pipeline that can hold different types of Effects!
Looking at the `pipeline.rs` file will give us insight into how
that's done.

```rust
...
pub struct Pipeline {
    fs: f32,
    effects: LinkedList<Box<dyn Effect + Send + Sync>>,
}
...
```

The `Pipeline` struct holds the effects in a `LinkedList`.

Lets unpack what `Box<dyn Effect + Send + Sync>` means.

The first part, `Box<...>`, means the data is stored on the heap.
The `LinkedList` can't hold the raw effect types since their size changes
depending on the type of effect it is.
This means it can't be known at compile time. To get around this issue,
we can encase them in a `Box<...>` which has a known size.

The `dyn Effect + Send + Sync` part means that the elements in this
`LinkedList` must implement `Effect`, `Send`, and `Sync`. But when we
looked at the `AllPass` struct, it didn't implement `Send` or `Sync`.
Why is the code able to compile?

The answer lies in auto trait implementations. When all the attributes
of a struct implement a given trait, the struct itself implements it
automagically. This means we don't have to go out of our way to
implement it manually.

Now that we understand the components of the struct, lets
look at the methods used to manipulate the effects loaded:

```rust
pub fn create(&mut self, effects: LinkedList<Box<dyn Effect + Send + Sync>>) {
  self.effects = effects;
}

...

pub fn tear_down(&mut self) {
  self.effects = LinkedList::new();
  self.effects.push_back(Box::new(AllPass));
}

...

pub fn run(&mut self, in_sample: f32, toggle: f32) -> f32 {
  if toggle <= 0. {
    return in_sample;
  }

  let mut out: f32 = in_sample;
  for effect in &mut self.effects {
    out = effect.run(out);
  }
  out
}

```

The `create` method sets the effects we use. 

The `tear_down` method removes
all the effects and puts an `AllPass` filter in their place. 

The `run` method ties all the effects together into one
coherent pipeline.

Lets start using `Pipeline` to create some sounds!

## Using The Pipeline Struct

```rust
fn new(plugin_info: &PluginInfo, _features: &mut ()) -> Option<Self> {
  let fs = plugin_info.sample_rate() as f32;
  let mut pline = pipeline::Pipeline::new(fs);

  // static example of filters in series
  let mut effects = LinkedList::new();
  effects.push_back(
    Box::new(lowpass::IIRLowPass::new(fs, 1000.)) as Box<dyn Effect + Send + Sync>
  );
  effects.push_back(
    Box::new(highpass::IIRHighPass::new(fs, 500.)) as Box<dyn Effect + Send + Sync>
  );
  effects.push_back(
    Box::new(echo::Echo::new(fs, 0.2, 10, echo::DecayType::Linear, 1.))
      as Box<dyn Effect + Send + Sync>,
  );
  pline.create(effects);
  Some(Self { pline })
}

fn run(&mut self, ports: &mut Ports, _features: &mut (), _: u32) {
  for (out_sample, in_sample) in Iterator::zip(ports.output.iter_mut(), ports.input.iter()) {
    *out_sample = self.pline.run(*in_sample, *ports.control);
  }
}
```

Here's a snippet of code from a plugin. Notice how we add
the `IIRLowPass`, `IIRHighPass`, and `Echo` to create a pass-band effect with an echo!

## The Next Steps

Now that we have the filters implemented and can use them in series,
the next step is to implement the concepts of the rules and instances
I talked about in earlier posts.

The representation of the data itself is not hard, using JSON will suffice.
The real complication is how to get the JSON data to the plugin and
allow for it to be updated as an artist sees fit.
One technique that seems to have promise is seting up a REST framework or
some other network based API.

The plugin can listen on localhost for incoming requests and set the rules and
instance wanted accordingly.
