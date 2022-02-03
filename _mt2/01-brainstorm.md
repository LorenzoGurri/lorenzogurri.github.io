---
layout: single
title: Music & Tech 2 Brainstorm
toc: true
toc_sticky: true
toc_label: "Ideas"
toc_icon: "file-alt"
excerpt: "What projects could I do?"
author: Lorenzo Gurri
---

In this post, I explore possible project ideas for my Music and Tech 2 class. The
ideal project is more heavy on programming in Rust.


## <i class="fa fa-linux"></i> &nbsp; Sound card driver (maybe in Rust?)

A sound card driver could be a pretty cool project. It would combine my interests
  of messing around in the linux kernel and (possibly) Rust. 

After doing research into sound card driver development, a project like this might
  be best without a semester time constraint. Writing an ALSA driver
  is a little messy and pretty [complex](https://www.kernel.org/doc/html/v4.17/sound/kernel-api/writing-an-alsa-driver.html).

Another limiting factor to this idea is if I decided to write it in Rust. The project
  that I would be taking advantage of, [Rust for Linux](https://github.com/Rust-for-Linux/linux),
  isn't stable enough to assure I could write a driver by the end of the semester.


## <i class="fa fa-headphones-alt"></i> A VST plugin effect in Rust

For this idea, I would create a VST plugin that can dynamically apply effects based 
  off of input data. For example, I could change the effects applied based off of the 
  current weather.

This project idea is much more manageable. There is already a [Rust crate](https://docs.rs/vst/0.2.1/vst/)
  for creating VST plugins aptly named `vst`. This project could be pretty 
  interesting for dynamic music generation.
