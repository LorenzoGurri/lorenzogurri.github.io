---
layout: single
title: Music & Tech 2 Filters
toc: true
toc_sticky: true
toc_label: "Filters"
toc_icon: "file-alt"
excerpt: "Writing filters in Rust"
author: Lorenzo Gurri
---

In this post I talk about implementing some simple filters
using the `lv2` library in Rust.

## Amplitude LFO

A Low Frequency Oscillator (LFO) is a super
useful tool for building cool sounding filters.
It takes a low frequency sinusoid (1-20Hz) 
and uses it to modulate something.
One easy way to start using them is to modulate 
the amplitude of the signal between 0 and 1. We
can do this pretty easily in Rust using the `lv2` crate.

### The LFO struct

Here we define and implement the `LFO` struct. We first define
our attributes of `sr` and `state`. The `sr` variable is
the sample rate of our audio. We get this from the host
when we initialize the plugin. This is only needed if we want
to oscillate in terms of seconds (aka we add a control port for
the user). The `state` variable will tell us where on the wave we are.

```rust
// Sinusoidal LFO
struct LFO {
  // sample rate
  sr: f64,
  // where we are on the wave
  state: f32
}

impl LFO {
  // create a new LFO object
  fn new(sr: f64) -> LFO {
      Self { sr, state: 1. }
  }

  // called on every sample.
  fn tick(&mut self, in_sample: f32, freq: f32) -> f32 {
    // since you can think of sinusoidal functions in
    // terms of circles, we only need the state to
    // increase from 0 to freq
    self.state = if self.state == freq {
      1.
    } else { 
      self.state+1.
    };
    
    // The sinusoid itself.
    (((2. * std::f32::consts::PI *self.state / freq).cos()+1.) / 2.) * in_sample
  }
}
```

The last line of the `tick` function isn't very clear. Lets write it
a little neater and see what it looks like on a graph.

{:refdef: style="text-align: center;"}
![formula](https://render.githubusercontent.com/render/math?math=\huge \frac{\cos(2\pi x / f)%2B1}{2})
{: refdef }

![Sinusoid](/assets/images/mt2_lfo_graph.png)

Where `x` is our `state` variable and `f` is `freq` variable that tells us
the frequency we want the LFO to work on. This still looks odd, lets break
it down further into two parts. The first part,
![formula](https://render.githubusercontent.com/render/math?math=\Large \cos(2\pi x / f)),
will give us a cosine wave of frequency `f`. The second part deals with 
shifting the output space of the cosine function. We want our output to
be between zero and one. The cosine function will give us values between
-1 and 1. So, we need to add one (0 to 2) and then divide by two (0 to 1)
to get the range we want.

We can now use our LFO struct with the `lv2` crate to make a simple plugin.
We could further develop it by adding a control port and allowing a user
to control the frequency, but we can do that later.

```rust
// not a real URL, just used for the example
#[uri("https://my.example.code/LFOfilter")]
struct LFOfilter {
    lfo: LFO
}

impl Plugin for LFOfilter {
  // the ports are the same as the previous example
  type Ports = Ports;
  type InitFeatures = ();
  type AudioFeatures = ();

  // create a new LFOfilter object, since it has an attribute
  // of type LFO, we need to initialize it.
  fn new(plugin_info: &PluginInfo, _features: &mut ()) -> Option<Self> {
    Some(Self {lfo: LFO::new(plugin_info.sample_rate())})
  }

  // run our lfo on the in-samples
  fn run(&mut self, ports: &mut Ports, _features: &mut (), _: u32) {
    for (out_sample, in_sample) in Iterator::zip(ports.output.iter_mut(), ports.input.iter()) {
      *out_sample = self.lfo.tick(*in_sample, self.lfo.sr as f32);
    }
    println!("{}", self.lfo.state);
  }
}

lv2_descriptors!(LFOfilter);
```
## TODO (MORE FILTERS)

