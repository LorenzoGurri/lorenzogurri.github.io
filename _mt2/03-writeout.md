---
layout: single
title: Music & Tech 2 Writeout
toc: true
toc_sticky: true
toc_label: "Writeout"
toc_icon: "file-alt"
excerpt: "Lets define what the plugin will do"
author: Lorenzo Gurri
---

In this post I sketch out the design for the plugin.

## Abstract View

If we think of the plugin as a black box, it has about 3 inputs and one output. The inputs are the **rules**, **instance**, and **audio**. 

{:refdef: style="text-align: center;"}
![Abstract view of plugin](/assets/images/mt2-03-abstract-design.png)
{:refdef}

The **rules** are the key value pairs that map some key with an effect. For example, if we want to map some weather to different effects, we could say **cloudy** pairs up with a low pass filter and **rainy** pairs up with a flanger. This may also include different parameters for the effects themselves.

The **instance** is putting these rules into action. Going back to the weather example, the instance of those rules might tell us the current weather, for example, cloudy.

The **audio** is just the track that we are affecting in the DAW.

In terms of our output, we have the input audio with the effects on it.

## Name

Obviously the most important part of creating a project is naming it something cool and maybe even tossing in some vector graphics for it. I decided to name the project `riaLV2` since it is based on in inputs of, you guessed it, the rules, the instance, and the audio. As for the vector graphics, I decided to go with books (like what rules are written in) and have the **i** for **instance** lean on the rules (since they depend on them).

{:refdef: style="text-align: center;"}
![Sick vector graphics](/assets/images/mt2-riaLV2-logo.png){: width="250" }
{:refdef}

## Implementation timeline

Creating a timeline is going to be a good way to stay on track and have a clear idea on the project's goals.


### March 4 (1 week allotment)
- Finish creating a bare bones environment plugin
- Setup a testing environment

### March 11 (2 week allotment)
- Start either implementation of effects or find a library

### March 25 (2 week allotment)
- Configure rule/instance

### April 1 (2 week allotment)
- Apply selected effects to audio buffer given rules/instances
