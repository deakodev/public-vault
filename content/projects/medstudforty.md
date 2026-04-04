---
title: "Medical Education Platform"
date: 2025-09-01
description: "A platform for medical students to study smarter and track progress."
tags: ["medicine", "education", "web"]
draft: false
---

Medical school produces an overwhelming volume of material with no good infrastructure for connecting concepts across disciplines. MedStudForty is a platform built to fix that — structured around active recall, spaced repetition, and cross-linking clinical content.

## The problem

Medical students spend enormous time re-reading passive notes rather than practicing retrieval. Existing tools (Anki, UWorld) optimize for isolated recall without supporting the synthesis that clinical reasoning actually requires. You need to know not just that a drug blocks a receptor, but how that connects to the presentation you'll see in the clinic.

## What it does

- Structured question banks organized by organ system and presentation
- Spaced repetition scheduling that adapts to individual weak areas
- Cross-linked concept maps so learners see connections across disciplines
- Progress dashboards that surface gaps rather than just scores

## Stack

Built with Next.js, PostgreSQL, and a custom spaced repetition algorithm. Designed with the constraint that it has to be fast enough to use between patients on a clinical rotation.

[GitHub](https://github.com/deakodev/medstudforty)
