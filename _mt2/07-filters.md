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

In this post I talk about implementing four filters
using the `lv2` library in Rust.

## Amplitude LFO

A Low Frequency Oscillator (LFO) is a super
useful tool for building cool sounding filters.
It takes a low frequency sinusoid (1-20Hz) 
and uses it to modulate something.
One easy way to start using them is to modulate 
the amplitude of the signal between 0 and 1. We
can do this pretty easily in Rust using the `lv2` crate.

### The LFO Struct

Here we define and implement the `LFO` struct. We first define
our attributes of `fs` and `state`. The `fs` variable is
the sample rate of our audio. We get this from the host
when we initialize the plugin. This is only needed if we want
to oscillate in terms of seconds (aka we add a control port for
the user). The `state` variable will tell us where on the wave we are.

```rust
// Sinusoidal LFO
struct LFO {
  // sample rate
  fs: f64,
  // where we are on the wave
  state: f32
}

impl LFO {
  // create a new LFO object
  fn new(fs: f64) -> LFO {
      Self { fs, state: 1. }
  }

  // called on every sample.
  fn tick(&mut self, in_sample: f32, freq: f32) -> f32 {
    // since you can think of sinusoidal functions in
    // terms of circles, we only need the state to
    // increase from 0 to freq-1
    self.state = (self.state+1)%(freq-1);
    
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
      *out_sample = self.lfo.tick(*in_sample, self.lfo.fs as f32);
    }
    println!("{}", self.lfo.state);
  }
}

lv2_descriptors!(LFOfilter);
```
## Simple Echo

An echo effect combines a number of delays and the input signal to
produce a, well, echo. To implement an echo, we need to have a
representation of a delay implemented.

### The Delay Struct

Here we define a simple delay. It populates a `LinkedList` with
samples until it is full and then starts returning them.

```rust 
struct Delay {
  fs: f32,
  buffer: LinkedList<f32>,
}

impl Delay {
  // create a new Delay object
  fn new(fs: f64) -> Delay {
    Self { fs: (fs as f32), buffer: LinkedList::new() }
  }

  // return either a delayed sample or the current one 
  // if the buffer isn't filled yet.
  fn tick(&mut self, in_sample: f32, delay: f32) -> f32 {
    if self.buffer.len() < (delay*self.fs) as usize {
      self.buffer.push_back(in_sample);
      return in_sample;
    }
    self.buffer.pop_front().unwrap_or(0.)
  }
}
```

To create the echo effect, we now combine a 
number of `Delay`s. It's worth noting that we add
a LV2 control port called `control`. This gives us the
delay the user wants in seconds and controls how large
the buffer gets in the `Delay` struct.

```rust
// Our Collection of LV2 ports
#[derive(PortCollection)]
struct Ports {
  control: InputPort<Control>,
  input: InputPort<Audio>,
  output: OutputPort<Audio>,
}

#[uri("https://my.example.code/DelayFilter")]
struct DelayFilter {
  // use 5 delays for the effect
  dly1: Delay,
  dly2: Delay,
  dly3: Delay,
  dly4: Delay,
  dly5: Delay,
}

impl Plugin for DelayFilter {
  type Ports = Ports;
  type InitFeatures = ();
  type AudioFeatures = ();

  fn new(plugin_info: &PluginInfo, _features: &mut ()) -> Option<Self> {
    Some(Self {
      dly1: Delay::new(plugin_info.sample_rate()),
      dly2: Delay::new(plugin_info.sample_rate()),
      dly3: Delay::new(plugin_info.sample_rate()),
      dly4: Delay::new(plugin_info.sample_rate()),
      dly5: Delay::new(plugin_info.sample_rate()),
    })
  }

  fn run(&mut self, ports: &mut Ports, _features: &mut (), _: u32) {
    for (out_sample, in_sample) in Iterator::zip(ports.output.iter_mut(), ports.input.iter()) {
      // combine the delays. As the echo dies, the amplitudes
      // of the delays get smaller and smaller.
      *out_sample = *in_sample
          + 0.75 * self.dly1.tick(*in_sample, *ports.control)
          + 0.50 * self.dly2.tick(*in_sample, *ports.control * 2.)
          + 0.25 * self.dly3.tick(*in_sample, *ports.control * 3.)
          + 0.10 * self.dly4.tick(*in_sample, *ports.control * 4.)
          + 0.05 * self.dly5.tick(*in_sample, *ports.control * 5.);
    }
  }
}
```

## Low Pass Filter

A low pass filter attenuates frequencies higher than a cutoff and allows
frequencies lower than it to pass. There are plenty of different types
but a very common one is the butterworth filter. The definition of an
`n` order butterworth filter's transfer function is:

{:refdef: style="text-align: center;"}
![formula](https://render.githubusercontent.com/render/math?math=\huge H(s)=\frac{1}{\sum_{k=0}^{n}\frac{a_k}{\omega_0^k}s^k})
{: refdef }

Note that this is in the frequency domain and is not discretized. We can get a given 
![formula](https://render.githubusercontent.com/render/math?math=\Large a_k),
through the definition below:

{:refdef: style="text-align: center;"}
![formula](https://render.githubusercontent.com/render/math?math=\huge a_{k%2B1}=\frac{\cos(k\gamma)}{\sin((k%2B1)\gamma)})
{: refdef }

Where 
![formula](https://render.githubusercontent.com/render/math?math=\Large \gamma=\frac{\pi}{2n})
and
![formula](https://render.githubusercontent.com/render/math?math=\Large a_0=1). As the order 
becomes larger, the butterworth filter's transition band gets smaller and the attenuation
becomes sharper.

Here we can see the bode plots of
![formula](https://render.githubusercontent.com/render/math?math=\Large n=2,4,8,16)
pole butterworth filters generated with the code below.

```python
import control
import matplotlib.pyplot as plt
import numpy as np

# code modeled from
# https://www.youtube.com/watch?v=HJ-C4Incgpw

# calculate the transfer function
# and the bode plot
def calc(n, dt, freq_c, omega_0):
    # calculate a_ks
    a = np.zeros(n+1)
    a[0] = 1
    gamma = np.pi / (2*n)
    for k in range(n):
        a[k+1] = (np.cos(k*gamma) / np.sin((k+1)*gamma))*a[k]

    # numerator and denominator of the transfer function
    num = np.array([1])
    den = np.zeros(n+1)

    # calculate the denominator
    for k in range(n+1):
        den[n-k] = a[k]/pow(omega_0,k)

    # continuous butterworth Transfer Function
    butterworth = control.TransferFunction(num,den)

    # calc the bode plot
    control.bode(butterworth, Hz=True)

    # graph
    fig = plt.gcf()
    mag_axis, phase_axis = fig.axes
    mag_axis.axvline(x=freq_c, color='red', linestyle='--')
    phase_axis.axvline(x=freq_c, color='red', linestyle='--')

# number of poles
n=2
# change in time (1 / sample rate)
dt = 1 / 44100
# cutoff frequency
freq_c = 400
# cutoff frequency in rad/s
omega_0 = 2 * np.pi * freq_c

calc(n, dt, freq_c, omega_0)
n=4
calc(n, dt, freq_c, omega_0)
n=8
calc(n, dt, freq_c, omega_0)
n=16
calc(n, dt, freq_c, omega_0)
plt.show()
```

![formula](/assets/images/mt2_butterworth.png)

Notice how the filter becomes better at attenuating the unwanted frequencies
as the number of poles increases
but the phase shift of the output signal generated becomes worse and worse.

To make this transfer function useful, we can first discretize it through
a bilinear transform:

{:refdef: style="text-align: center;"}
![formula](https://render.githubusercontent.com/render/math?math=\huge s = \frac{2}{T}\cdot\frac{1-z^{-1}}{1%2Bz^{-1}})
{: refdef }

Afterwards, we turn the transfer function into a difference equation we could
use programmatically.

{:refdef: style="text-align: center;"}
![formula](https://render.githubusercontent.com/render/math?math=\huge H(z) = \frac{\beta_0 %2B \beta_1z^{-1}%2B\beta_1z^{-2}%2B\dots}{1-\alpha_1z^{-1}-\alpha_2z^{-2}-\dots})
{: refdef }

{:refdef: style="text-align: center;"}
![formula](https://render.githubusercontent.com/render/math?math=\huge y[n]=\alpha_1y[n-1]%2B\alpha_2y[n-2]%2B\dots%2B\beta_0x[n]%2B\beta_1x[n-1]%2B\dots)
{: refdef }

### The IIRLowPass Struct

In Rust, there is a library that is capable of doing
these operations for us called `biquad`.

It is able to calculate 2nd order IIR Filters.
```rust
struct IIRLowPass {
    fs: f32,
    cutoff: f32,
    bq: DirectForm2Transposed<f32>,
}

impl IIRLowPass {
  fn new(fs: f64) -> IIRLowPass {
    let coeffs = Coefficients::<f32>::from_params(
      Type::LowPass,
      (fs as f32).hz(),
      (20000 as f32).hz(),
      Q_BUTTERWORTH_F32,
    )
    .unwrap();

    Self {
      fs: (fs as f32),
      cutoff: 20000.,
      bq: DirectForm2Transposed::<f32>::new(coeffs),
    }
  }

  fn tick(&mut self, in_sample: f32, cutoff: f32) -> f32 {
    if cutoff != self.cutoff {
      let coeffs = Coefficients::<f32>::from_params(
        Type::LowPass,
        (self.fs as f32).hz(),
        (cutoff as f32).hz(),
        Q_BUTTERWORTH_F32,
      )
      .unwrap();
      self.bq = DirectForm2Transposed::<f32>::new(coeffs);
      self.cutoff = cutoff;
    }
    self.bq.run(in_sample)
  }
}
```

`Coefficients::<f32>::from_params(...)` calculates the coefficients
for the type of filter, the sample rate, and the cutoff frequency.

Every tick, the coefficients and previous two output values are used
to calculate the next output sample. Also note that the control port
is in hz from range 20 to 20000.

```rust
#[uri("https://my.example.code/LPFilter")]
struct LPFilter {
  lpf: IIRLowPass,
}

impl Plugin for LPFilter {
  type Ports = Ports;
  type InitFeatures = ();
  type AudioFeatures = ();

  fn new(plugin_info: &PluginInfo, _features: &mut ()) -> Option<Self> {
    Some(Self {
      lpf: IIRLowPass::new(plugin_info.sample_rate()),
    })
  }

  fn run(&mut self, ports: &mut Ports, _features: &mut (), _: u32) {
    for (out_sample, in_sample) in Iterator::zip(ports.output.iter_mut(), ports.input.iter()) {
      *out_sample = self.lpf.tick(*in_sample, *ports.control)
    }
  }
}
```

## High Pass Filter

In terms of math, we can represent a butterworth high pass
filter as a low pass filter transformed with
![formula](https://render.githubusercontent.com/render/math?math=\Large s=\frac{\omega_0^2}{s})

Here we can see the bode plots of
![formula](https://render.githubusercontent.com/render/math?math=\Large n=2,4,8)
pole high-pass butterworth filters. It was generated with the same python code
from the low-pass filter with the exception of the below snipet that
calculates the the sum of the denominator with the transformed 
![formula](https://render.githubusercontent.com/render/math?math=\Large s).

```python
# high pass
s = pow(omega_0, 2) / control.TransferFunction.s

# sum denominator
s_den = np.int64(0)
for k in range(n+1):
  s_den += den[n-k]*(pow(s,k))

H = 1 / s_den

# continuous butterworth Transfer Function
butterworth = control.TransferFunction(H)
```

![formula](/assets/images/mt2_butterworth_high.png)

In terms of implementing the filter, all we really need to change
is the type of filter used:

```rust 
let coeffs = Coefficients::<f32>::from_params(
  Type::HighPass,
  (self.fs as f32).hz(),
  (cutoff as f32).hz(),
  Q_BUTTERWORTH_F32,
).unwrap();
```

