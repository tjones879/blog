---
title: "NES Architecture"
date: 2018-05-31T20:18:00-06:00
draft: "true"
---

As many people are aware, the NES was released in the early 1980's,
and as a result had fairly simple hardware. These are some of the
important components to consider:

  - A modified **MOS 6502** running at 1.79MHz, (similar to the
    Commodore 64 sans binary coded decimal).
  - An **audio processing unit** (APU) in the RP2A03
  - A **picture processing unit** (PPU) in the form of the 2C02
  - Separate 16kB address spaces in the CPU and PPU with support for
    about 200 mappers that are responsible for translating the
    graphical and program memory into the above address spaces 
    
Each of these will be further explored independently both in this post
and their possible respective design posts.

## CPU - 6502

The MOS 6502 is an 8-bit processor with only 6 special-purpose registers
and, as stated above, a 16kB addressable memory space. What makes this
processor fairly interesting is the presence of several addressing modes:

- Accumulator:  Many instructions can operate directly on the accumulator
- Immediate:    Values prepended as `#<value>` will use the value itself instead
                of an address
- Zero page:    The value is fetched as an 8-bit address within the zero page
- Absolute:     The value is fetched as a 16-bit address anywhere in memory
- Relative:     Branches can supply an address relative to the current program counter (PC)
- Indirect:     JMP can jump to the address stored in a 16-bit pointer in memory

This is interesting from a design perspective because otherwise identical operations
may require a different number of clock cycles depending on its addressing mode.
You may notice in later posts that the CPU's address space has multiple mirrors
to other locations, meaning that the 64KB memory space is not fully utilized.

Luckily, the 6502 is an old enough processor to not require many of the
advanced techniques required even for the Gamecube or Nintendo Wii.

## APU - RP2A03

The audio processor is a very simple system to emulate. It consists of only
5 audio channels, each mostly composed of 3 or 4 8-bit registers that define
their behavior. It is unfortunately not too interesting to observe from a high
level perspective. Expect an indepth post covering the far more interesting
Rust implementation details at a later date.

## PPU - 2C02

The PPU on the NES is arguably one of the most interesting components in the
system. It features its own 16kB of memory space separate from the CPU. The
mappers found on the cartridge are responsible for directing where each segment
of the address space actually resides.

## ROM Formats - iNES

iNES is the standard format for NES ROM's. It consists of a very simple 16 byte
header followed by all PRG and CHR ROM data. The file format for ROM's is simple
enough by itself, but emulating the actual cartridges for the NES is deceptively
complex.

Officially, the NES could support 16KB or 32KB of PRG ROM and 8KB of CHR ROM. Anything
beyond these resources required additional hardware to implement features manually.
As a result, there are dozens of popular implementations of cartridge hardware.

The [PowerPak](https://nerdlypleasures.blogspot.com/2016/03/nes-powerpak-and-everdrive-n8-mapper.html)
is the most extreme example of this madness. It allows users to insert a Compact Flash
card formatted as FAT32 and load dozens of games among other features.

Many modern emulators also support submappers for common mappings that correctly emulate
sometimes faulty or buggy behavior in specific cartridges.

## Resources

- Much of the information here can also be found on the wonderful [nesdev wiki](http://wiki.nesdev.com).
