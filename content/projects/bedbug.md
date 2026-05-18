---
title: "3D Rendering Engine"
date: 2024-11-01
description: "A medical visualization engine built in C and Vulkan for rendering anatomical models and DICOM imaging data."
tags: ["graphics", "c", "vulkan", "dicom", "medicine", "systems"]
ongoing: true
draft: false
---

Bedbug is a real-time visualization engine built from scratch in C99 and Vulkan, targeting anatomical model rendering and medical imaging data for educational and diagnostic applications.

## Why build this

Medical education and diagnostics both depend on spatial understanding of three-dimensional anatomy. Most existing tools either lock you into expensive proprietary software or sacrifice real-time interactivity for data fidelity. Building a renderer from scratch (with a graphics pipeline I own entirely) means I can optimize specifically for this domain: variable-resolution anatomical meshes, multi-planar DICOM reconstruction, and overlays that make sense for clinical reading rather than gaming.

## What it does

- Vulkan-based rendering pipeline for real-time performance on anatomical geometry
- DICOM file parser and interactive viewer for exploring medical imaging datasets
- Support for loading and rendering anatomical meshes for spatial study
- Designed around educational and diagnostic display requirements, not game engine conventions

## Stack

C99, Vulkan, Bash

[GitHub](https://github.com/deakodev/bedbug)
