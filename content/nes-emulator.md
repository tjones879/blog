---
title: "Building a NES Emulator"
date: 2018-05-31T19:14:00-06:00
draft: "true"
---

I've recently gotten into learning how to write emulators. It is generally
recommended for newcomers to start by writing a [CHIP8](https://en.wikipedia.org/wiki/CHIP-8)
emulator. This is a simple 8 bit virtual machine that demonstrates many
components of simple emulator design.

For experienced developers, CHIP8 should not pose much of a problem. Popular
systems to move onto are the very popular NES and GBA. These are still
relatively simple systems but start to pose actual design requirements. Being
based on real hardware systems, these consoles pose a more difficult problem
than CHIP8.

I chose to build a NES emulator after remembering many of my favorite classic
games on that platform.

I am also taking this as an opportunity to teach myself Rust. While becoming
familiar with ANSI C and modern C++, I recognized a great situation to learn
Rust.

This blog series will cover my adventures in developing [RustiNES](https://github.com/tjones879/RustiNES),
one of the several NES emulators written in Rust. These posts will also help
serve as a reference for myself and hopefully others.

## Table of Contents

1. [NES Architecture]({{< ref "nes-architecture.md" >}})

## Resources
