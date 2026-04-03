---
title: "Designing for uncertainty"
date: 2026-02-18
description: "Clinical decisions are rarely about missing information. They're about acting under irreducible uncertainty."
tags: ["hci", "medicine", "research"]
draft: false
---

A common framing in clinical decision support is the information problem: the physician doesn't have what they need, so the system should surface it. If we can just get the right data in front of the right person at the right time, better decisions follow.

This framing is wrong, or at least incomplete.

## Most decisions aren't data-limited

In my experience at the bedside, the hard calls aren't usually the ones where I lack information. They're the ones where I have a lot of information that points in different directions, and I need to act anyway — often quickly, often irreversibly.

The cognitive challenge isn't retrieval. It's integration and commitment under uncertainty.

A system that surfaces more data can actually make this harder. More signals means more to reconcile. A well-designed alert can become noise when it fires too often. The interface that seemed helpful in a controlled study becomes a liability in a chaotic shift.

## Uncertainty as a design variable

What would it look like to design for uncertainty rather than against it?

A few things I keep coming back to:

**Calibration over confidence.** Decision support tools tend to present outputs as answers. A probability estimate that says "82% chance of sepsis" is treated as a fact rather than a distribution. What if the interface instead communicated the model's confidence interval, or flagged the cases it tends to get wrong?

**Legible reasoning.** When I disagree with a recommendation, I want to understand why it was made — not to override it, but to integrate it with what I know that the system doesn't. Black-box tools fail here. The path to trust isn't accuracy alone; it's transparency about the basis for the output.

**Supporting commitment, not just deliberation.** Eventually you have to act. Interfaces that endlessly surface options without supporting the move from analysis to decision aren't serving the clinician — they're serving the designer's discomfort with the messy reality of practice.

## What this means for research

I'm trying to build this into how I design studies. The interesting question isn't "did the system give the right answer?" It's "how did the system change the way the clinician reasoned, and was that change adaptive?"

That's harder to measure. But it's closer to the actual problem.
