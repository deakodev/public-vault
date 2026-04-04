---
title: "3D Rendering Engine"
date: 2024-11-01
description: "A software 3D renderer written from scratch in C."
tags: ["graphics", "c", "systems"]
draft: false
---

Bedbug is a software rasterizer built from scratch — no OpenGL, no GPU. It implements the full pipeline: perspective projection, triangle rasterization, z-buffering, texture mapping, and a basic lighting model. Written in C.

## Why build this

I wanted to understand what graphics hardware is actually doing. Using a library like OpenGL abstracts away the math in a way that works for shipping products but leaves gaps if you want to understand what's happening. Writing a renderer from scratch forces you to confront every step: how a 3D point becomes a 2D pixel, how you determine which surfaces are visible, how lighting interacts with geometry.

## What it does

- Perspective-correct texture mapping
- Flat and Gouraud shading
- Z-buffer for depth sorting
- Loads and renders OBJ files
- Runs in a software framebuffer (no GPU required)

The name is a nod to the bugs you inevitably find when you're hand-rolling matrix math at 2am.

[GitHub](https://github.com/deakodev/bedbug)
