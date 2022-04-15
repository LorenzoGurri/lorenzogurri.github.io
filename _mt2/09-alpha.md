---
layout: single
title: Music & Tech 2 Alpha
toc: true
toc_sticky: true
toc_label: "Alpha"
toc_icon: "file-alt"
excerpt: "Alpha 0.0.1 release"
author: Lorenzo Gurri
---

In this post I talk about the alpha release of the riaLV2 project!

# Dynamic Effect switching

The entire premise of this plugin is its ability for its effects
to be switchable through external means. For example, swapping
between effects based on different keys we press. To do this,
we need the plugin to be able to communicate with the outside
world in some way. As an initial approach, the plugin can set its
effects through a constantly read configuration file.

The LV2 framework has the ability to support this approach. Through
worker threads, we can schedule non-realtime work to be done. In this
case, reading the config file.

Having the effects swapped through a central config presents some
downsides and limitations. For example, there can be only one plugin
instance running at once. Another downside is runtime. The file is
opened and polled every `work()` call. This adds variable latency in
the application of the effects.

# Steps Towards Beta

To see the most up to date version of what the beta release needs,
take a look at the milestone setup on GitHub 
[here](https://github.com/LorenzoGurri/ria_lv2/milestone/1).

Some of the more notable issues that will be addressed are:

- **A new project name**: The initial concept of rules and instances never
really came to fruition. The plugin just takes what effects a user wants
and applies them.

- **Removing In-House Effects**: Making our own effects is great for
learning, but bad for for flexibility. Being able to load in your
own effects would be much more useful. Shifting the design of the
project to a plugin host that can be loaded into a DAW is much more
practical for a user.
