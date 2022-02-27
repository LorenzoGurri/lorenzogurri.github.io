---
layout: single
title: Music & Tech 2 Deciding
toc: true
toc_sticky: true
toc_label: "VST Research"
toc_icon: "file-alt"
excerpt: "Deciding the project and doing a little research"
author: Lorenzo Gurri
---

In the previous post, I outlined some ideas for a project idea. I plan to move
forward with writing a plugin, however I decided to write it using the LV2 format
vs VST. But why Rust and why LV2 over VST?

## <i class="fab fa-rust"></i> The Rust Programming Language

Rust is a powerful programming language that helps you write faster, more
reliable software[^1]. Since Rust is extremely fast, it makes it optimal
for audio development. Rust gets its speed through a bunch of different means.

One of which is zero-cost abstractions. This means you get to use abstract 
coding practices like using generics or collections (vectors, hashmaps, etc)
without having to worry about them coming to bite you in the butt with a slower
runtime later.

Another is the Rust borrow checker. When writing a program in C or C++,
memory errors are one of the most sidious and obnoxious errors out there.
It's too easy to dereference a pointer and run into a dreaded segmentation fault or
use after free bug. It's also a pain to keep track of pointers and make sure that data that
isn't needed anymore is freed at the right time. Many programming languages handle this
through a garbage collector but Rust does it differently (and more efficiently). 

Enter the borrow checker. Rust's borrow checker is able to make sure your program
is free from these memory errors. It does this though a set of rules a programmer must follow.
One of the most notable is probably that it won't let you have multiple mutable references to
data at the same time. This rule follows under the idea of Rust lifetimes. [Lifetimes](https://doc.rust-lang.org/rust-by-example/scope/lifetime.html) allow Rust
to keep track of references. Below is an example that you can test yourself online within the
[Rust playground](https://play.rust-lang.org/)

```rust
fn main() {
    // When a variable is mutable, it can be manipulated
    let mut x = String::from("Hello World!");           
                                         
    /* Get a mutable reference to x
     * This means we can change the variable x through the variable borrow_x
     */             
    let borrow_x = &mut x;
    /* If we try to get another reference, Rust won't let us
     * Having more than 1 mutable reference in scope is aginst Rust's rules
     */
    // let borrow_x_again = &mut x;

    // x is now equal to "Hello Borrowed!"
    *borrow_x = String::from("Hello Borrowed!");
    
    // Print out our new value for x
    println!("{}", x);
}
```

The obvious alternative to using Rust for this project is to use C or C++ and the [JUCE framework](https://github.com/juce-framework/JUCE). 
This is a tride and true way of writing a plugin and has a lot of good documentation. For this
project however, I want to create something that might draw people into exploring Rust.
Rust is lesser known but has a lot of cool features that I believe can make it a powerful tool.

## <i class="fas fa-volume-up"></i> VST and LV2 plugins in Rust

Virtual Studio Technology (VST) is a plugin format for digital audio workstations.
It allows for an artist to add effects or instruments. The VST format is supported 
by Windows, MacOS, as well as Linux. The LV2 format offers similar compatibilities,
however, isn't nearly as popular as VST.

If VST is so popular compared to LV2, why use LV2? The answer is its support in Rust.
The current library for creating VST plugins (aptly named [vst](https://docs.rs/vst/0.2.1/vst/#modules))
is much more limited than that of the LV2 library (also aptly named: [lv2](https://docs.rs/lv2/latest/lv2/)). 
Using LV2 offers real-time compatible plugins as well as much better documentation[^2].
The library also offers the ability save the state of the plugin.

[^1]: See [The Rust Book](https://doc.rust-lang.org/stable/book/ch00-00-introduction.html)
[^2]: The [Rust-LV2 Book](https://rustaudio.github.io/rust-lv2/), [API Docs](https://docs.rs/lv2), and the [LV2 spec reference](https://lv2plug.in/ns/)
