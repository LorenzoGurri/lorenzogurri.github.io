---
layout: single
title: Music & Tech 2 VST Research
toc: true
toc_sticky: true
toc_label: "VST Research"
toc_icon: "file-alt"
excerpt: "VST plugin development in Rust"
author: Lorenzo Gurri
---

<!--Write about:
- VST framework
  - compare to others
  - benefits
  - drawbacks
- VST in Rust
  - libraries
  - how good are they -->


In this post, I give a brief intro of Rust, the VST interface, and how they can fit together.

## <i class="fab fa-rust"></i> The Rust Programming Language

Rust is a powerful programming language that helps you write faster, more
reliable software[^1]. Rust does this through a bunch of different means.

One of which is zero-cost abstractions. This means when you write a program 
in Rust, you don't have to worry about abstractions slowing you down.

Another is the Rust borrow checker. When writing a program in C or C++,
there are many bugs related to memory. Rust mitigates these bugs through
memory safe guarantees that can be made at compile time.

## <i class="fas fa-volume-up"></i> VST plugins in Rust

Virtual Studio Technology (VST) is a plugin format for digital audio workstations.
It allows for an artist to add effects or instruments. There are other formats
such as AU, AAX, RTAS, or TDM but VST is by far the most popular[^3]

In Rust, the `vst` library can be used to create VST plugins while taking advantage of
Rust's powerful features. While it's currently not fully-featured (we can't make a
graphical user interface through it), we can take advantage of other libraries to handle that.


[^1]: See [The Rust Book](https://doc.rust-lang.org/stable/book/ch00-00-introduction.html)
[^2]: Nice article on [memory safety in Rust](https://www.heapoverflow.cn/en/post/Rust-memory-safety.html)
[^3]: Comparing different [audio formats](https://www.electronicdrumadvisor.com/plugin-formats-differences-between-vst-vst3-au-aax-rtas-tdm/)
