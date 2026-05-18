---
title: "Medical Education Platform"
date: 2022-09-01
description: "A full-stack USMLE/COMLEX board prep platform built for medical students."
tags: ["medicine", "education", "web"]
ongoing: false
draft: false
---

MedStudForty is a full-stack platform built to improve preclinical diagnostic reasoning through USMLE and COMLEX examination-style practice. Built from 2022 through 2026 alongside active tutoring work, it was iterated against real student performance data.

## What it does

- Timed clinical question blocks emulating USMLE and COMLEX examination formats
- Real-time testing interface with integrated user authentication and session analytics
- Custom content management system for question authoring and editing
- Instructor dashboard for tracking individual student performance and identifying weak areas
- SEO-optimized delivery so students can find topic-specific resources organically

## What drove the design

Existing board prep tools (UWorld, Amboss) are closed systems. Building the platform myself meant I could instrument it exactly, tracking not just whether a student got a question right, but how long they spent, where they hesitated, and which conceptual gaps kept reappearing. That data shaped both the tutoring and the question bank.

## Stack

TypeScript, React/Next.js, MySQL

[GitHub](https://github.com/deakodev/medstudforty)
