---
title: "Stereo Coupling"
date: 2017-08-09T11:26:12-06:00
draft: "true"
---

*This is part of a series of posts designed to document the processes I
followed to improve FFmpeg's native vorbis encoder.*

Properly encoding multi-channel audio poses a unique opportunity to
implement techniques that can provide very serious memory savings in our
output stream.

The basic question we'd like to answer is as follows:

    How do we encode multiple channels of audio while minimizing both
    redundancy and perceivable error?

We will compare all possible solutions against encoding each channel as a
separate entity i.e. **Simple Stereo**. This means that no coupling or
interleaving will occur. Some benefits of this approach include:

* Simple computationally -- Neither the encoder or decoder have to complete
  costly calculations to produce a usable output.
* Simple to understand -- There is no additional complexity due to the stereo
  coupling mechanism. Complexity is one of the major hurdles for developers new
  to media coding.
* Guaranteed lossless coding

This is offset by the large amount of redundancy that occurs. We will later
see the savings potential of different approaches. This is our bubble sort
of stereo encoding.

### A review of stereo encoding

#### Intensity Stereo

With this model, we treat all samples as a sum of their parts and merge
both channels into one. In order to preserve some of the information stereo
audio provides, we additionally transmit data that determines how to pan
the frequency of these samples.

### Mid/Side Stereo

### Phase Stereo

*FFmpeg is an open source software. Being such, if you'd like to see the actual
implementation of this, feel free to reference commits: SOURCE, SOURCE, SOURCE.*

*Have questions, comments, or suggestions? As usual, please feel free to contact
me via email or social media above.*
