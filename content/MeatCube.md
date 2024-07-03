+++
title = "Unreal Engine Chaos Flesh Test Project - Meat Cube Re-creation"
date = "2023-11-15"
description = "A short R&D project investigating the new Chaos Flesh plugin for Unreal Engine 5.3."
[taxonomies]
tags = ["Unreal Engine", "Physics", "Chaos"]
+++

A short R&D project to investigate the new experimental Chaos Flesh plugin for Unreal Engine 5.3.

The primary goal of this project was to re-create the infamous Meat Cube from the 2008 GDC demo of Unreal Engine 3 (see link below). The secondary goal was to evaluate the capabilities, ease of setup and runtime performance of the Chaos Flesh plugin.

The results were positive, and the tetrahedral simulation does function in a real-time context. There were, however, quite a few limitations:

- The simulations do not interact with other objects in the scene, beyond simple collisions (which need to be registered somewhat manually). This notably includes friction, which is key to making the simulation realistic when the flesh body rests any weight on other colliders. I was able to emulate this with physics dampening somewhat, but it is far from perfect.
- The flesh asset does not support variability in the flesh properties. This means you can not have some sections of the flesh be more tense or gelatinous. I don't expect that's something super necessary to the intended use cases for this tech, however, it does limit this (very niche!) use case.

It is worth noting that the Chaos Flesh plugin is still experimental, and Epic plans to continue to improve it until it is production-ready.
