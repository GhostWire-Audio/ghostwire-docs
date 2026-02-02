# GhostWire Audio  
*A one-person lab for real-time sound systems*

GhostWire Audio exists because sound is emotional, but the machines that move it are mechanical—and somewhere in between, there’s a gap that most people never get to see.

This is my attempt to live in that gap.

---

## What This Is

GhostWire is a personal, open-source audio systems lab. I use it to build and study the invisible parts of sound software: real-time engines, DSP graphs, parameter systems, testing tools, and experimental plugins.

Some projects here are polished. Some are prototypes. Some are just questions written in C++ and Python.

Everything is documented, readable, and meant to be learned from.

---

## What I Care About

- **Real-time safety**  
  No glitches, no hidden locks, no surprise allocations in the audio thread. If something can break sound, I want to understand exactly how and why.

- **Systems over features**  
  I’m more interested in how things are built than how many buttons they have.

- **Transparency**  
  If a tool works, you should be able to trace it from the UI all the way down to the buffer and the math.

- **Curiosity first**  
  This lab exists to explore, not to optimize a product funnel.

---

## How This Lab Is Structured

- **gw-core**  
  The engine room. A real-time, cross-platform C++ audio engine and DSP framework.

- **gw-testlab**  
  The instruments. Python and C++ tools for measuring, visualizing, and validating DSP systems.

- **spectral-ghost**  
  The public prototype. A real plugin that turns engine code into something people can touch and hear.

- **ghostwire-docs**  
  The notebook. Architecture, design decisions, and long-form explanations of what I’m learning.

---

## Why This Exists

Most creative software hides its machinery. That makes it easy to use—but hard to understand.

GhostWire is my way of pulling the covers back and asking:
- What does “real-time” actually mean in code?
- Where do performance tradeoffs show up in system design?
- How does signal theory survive contact with operating systems, drivers, and hardware?

If something here is confusing, that usually means I don’t understand it well enough yet.

---

## Status

This is an active, evolving lab. Things will change. APIs will break. Ideas will get replaced.

That’s part of the point.

---

## License & Use

Everything here is open-source. If you find something useful, take it apart, remix it, and build something better.

If you have questions, ideas, or corrections, open an issue. Curiosity is welcome.
