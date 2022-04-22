---
layout: single
title: Music & Tech 2 Beta Timeline
toc: true
toc_sticky: true
toc_label: "Beta Timeline"
toc_icon: "file-alt"
excerpt: "The Road To Beta"
author: Lorenzo Gurri
---

In this post I talk about a timeline for the beta release.

# Release Requirements

I touched upon the release requirements lightly in the previous post.
Here I'll solidify them and create a timeline for the beta release.

- **Unit Testing:** Testing throughout the project helps keep code
consistent and bug free.

- **Add Stereo Support:** Currently, the plugin only support mono audio.
Adding stereo is a prerequisite for other requirements.

- **Remove Homemade Effects:** To expand the functionality of the plugin,
the homemade effects should be removed. Instead, the plugin should
be able to host other plugins that act as effects.

- **Switch To Network Based Approach:** The plugin shouldn't read from
a file to make the effects dynamic. Instead, it should listen on a
port.

- **Rename:** The original name of the project isn't reflective
of the actual implementation. In previous posts, the idea of 
rules and instances never came to fruition.

- **Documentation:** Usage documentation is super important.

# Timeline

Throughout the timeline, unit and integration tests are always being tweaked.

- Stereo Support (1 week)
- Remove Homemade Effects (3 weeks)
- Switch To Network Based Approach (3 weeks)
- Rename & Usage Documentation (2 weeks)
