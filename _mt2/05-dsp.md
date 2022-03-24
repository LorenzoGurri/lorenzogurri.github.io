---
layout: single
title: Music & Tech 2 DSP
toc: true
toc_sticky: true
toc_label: "DSP"
toc_icon: "file-alt"
excerpt: "Time to learn about DSP"
author: Lorenzo Gurri
---

In this post, I explore the basics of Digital Signal Processing.

## DSP Basics

The world of digital signal processing is really interesting.
It's a backbone to many different technologies like telecom,
audio processing, image processing, and echo location. Without it,
we wouldn't have radar, MRIs, space photography enhancements,
earthquake recording, and a whole lot of other things we take
for granted.

To strart to understand DSP, we need to understand some basic signal
terminology.

### Signal
A signal is like a function. It shows how one variable
varies with respect to another. For example, how amplitude changes
over time.

### Continuous Signal
A continuous signal is one that deals in an infinite space.

### Discrete Signals
A discrete signal is one that deals in a finite space.

### Sample Number
A sample number is an index in an array of samples.

### Quantization
Quantization is when a continuous signal is converted into
a discrete signal. The way to do this is through sampling.
Every certain number of Hz, we can "sample" the value of the
signal, this will give us a finite number of samples that we
can use to represent a continuous signal in a computer.

### Aliasing
One issue that can arise when sampling is aliasing. This occurs
when the signal's frequency is over half of the sampling rate
and the samples start to resemble the wrong frequency.

### Antialias Filters

An antialiasing filter's job is to filter out the parts of a signal
that are higher than half the sample rate (The Nyquist-Shannon sampling
theorem). 
Three very important filters that do this job are the Butterworth,
Chebyshev, and Bessel filters. Each have their pros and cons. For example,
the Chebyshev filter optimizes the drop in amplitude (rolls-off). The
Butterworth filter is optimized for passband (the part of the signal
allowed through the filter) flatness. The Bessel filter optimizes 
step responce which is when a signal quickly changes value. Of the
three, the Butterworth offers the best balance.


Statistics and probability are also big parts of DSP. For example,
it might be worth while to know the distribution of sample values or
the probability of a given sample being between two values.

### Mean: ![formula](https://render.githubusercontent.com/render/math?math=\Large\mu=\frac{1}{N}\sum_{i=0}^{N-1}x_i)

The mean tells us the average value of a signal by summing the samples
together and dividing by the sample number

### Standard Deviation: ![formula](https://render.githubusercontent.com/render/math?math=\Large\sigma=\sqrt{\frac{1}{N-1}\sum_{i=0}^{N-1}(x_i-\mu)^2})

The standard deviation tells us how each sample deviates from the mean.

### Variance: ![formula](https://render.githubusercontent.com/render/math?math=\Large\sigma^2)

The variance tells us the power of the fluctuation in the standard deviation.

## Linearity

Linearity is a concept that opens the door to the foundational concepts
of digital signal processing: superposition and decomposition.

There are two requirements for linearity and one soft requirement.

### Homogeneity
This says that given an input ![formula](https://render.githubusercontent.com/render/math?math=\Large x[n])
and its corresponding output ![formula](https://render.githubusercontent.com/render/math?math=\Large y[n]):

{:refdef: style="text-align: center;"}
![formula](https://render.githubusercontent.com/render/math?math=\huge kx[n]\Rightarrow ky[n]).
{: refdef}

### Additivity
This says that given an inputs ![formula](https://render.githubusercontent.com/render/math?math=\Large x_1[n],x_2[n])
and their corresponding outputs ![formula](https://render.githubusercontent.com/render/math?math=\Large y_1[n],y_2[n]):

{:refdef: style="text-align: center;"}
![formula](https://render.githubusercontent.com/render/math?math=\huge x_1[n]%2Bx_2[n]\Rightarrow y_1[n]%2By_2[n]). 
{: refdef}


### Shift Invariance
This says that given an input ![formula](https://render.githubusercontent.com/render/math?math=\Large x[n])
and its corresponding output ![formula](https://render.githubusercontent.com/render/math?math=\Large y[n]):

{:refdef: style="text-align: center;"}
![formula](https://render.githubusercontent.com/render/math?math=\huge x[n%2Bs]\Rightarrow y[n%2Bs])
{: refdef}

## Decomposition

The inverse to synthesis, decomposition deals with breaking apart
signals into component parts that can then be summed back together
to create the original signal.

## Superposition

Superposition is the heart of digital signal processing. It takes the
ideas of linearity and decomposition and combines them.

Consider a signal ![formula](https://render.githubusercontent.com/render/math?math=\Large x[n]) 
decomposed into M parts such that
![formula](https://render.githubusercontent.com/render/math?math=\Large x[n]=x_0[n]%2Bx_1[n]%2B\dots%2Bx_M[n])

Now pass each input signal component through the system to create the output signal components
![formula](https://render.githubusercontent.com/render/math?math=\Large y_0[n]%2Bx_1[n]%2B\dots%2Bx_M[n]).
We can now sum these component parts to create the output signal
![formula](https://render.githubusercontent.com/render/math?math=\Large y[n])

The value of superposition is its ability to break apart more complicated signals into easier component parts.
