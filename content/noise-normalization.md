---
title: "Noise Normalization"
date: 2017-08-08T09:26:19-06:00
draft: true
---

*This is part of a series of posts designed to document the processes I
followed to improve FFmpeg's native vorbis encoder.*

An identifiable characteristic of many lossy codecs is the noise signature
produced at low bitrates when pyschoacoustic transparency can't be easily
achieved.

Early specifications of [MPEG3]() often resulted in a metallic effect when
attempting to code at too low of bitrates. This is due to a difference in
scale between error due to quantization and human perception.

Vorbis handiles this in a different manner called **Noise Normalization**.
While it seems like a fairly generic term, it decomposes into a relatively
simple process. Quantization of the floor curve makes it possible for
energy to be lost in certain logarithmic bands that roughly map to how we
perceive sound. In order to maintain this, the vorbis encoder will attempt
to promote 

If this isn't clear to you, imagine the following situation where we want
to compress the following values so that they can be stored in fewer bits:

Take the array $a = [1.473, 5.972, 2.368, 3.714, 6.052, 2.863]$ and
assume these are all stored in memory as IEEE floating point numbers.
We can store these values within an error of <1E-4, but each would
require 32 bits. Let's entertain the notion of sacrificing
precision for significant memory savings.
because we are only using the range $[1, 6]$, why leave room for
256 possible values?

Our quantization has lowered the memory requirements from
$sizeof(a) = 24 bytes$ to $sizeof(a) = $ and only introduced an
error of $1E-4$.

These memory savings are incredible and will not produce audible artifacts
as long as our error is less than the [ATH](a.com), or the [noise mask]() in the
case of vorbis. These are very significant savings for the little amount of
work required of us. However, as our target bitrate lowers, more
quantization error will inevitably occur.

This is the source of metallic noise in mp3 and the gaussian noise produced
by libvorbis.

The solution provided by the libvorbis encoder is actually quite clever and
produces much more pleasing error. After the floor curve has been encoded,
the coder will actually decode the generated values, as the decoder would,
to obtain integer masking values. With these values, we can more closely
manipulate what we want our output to look like.

If we assume that all values are encoded in the floor, we can look up the
energy for each quantized value in the inverse dB lookup table. We then
calculate the sum of squares for an error estimate.


After the floor curve is calculated, the vorbis encoder
will attempt to maintain a constant overall energy for each band.

### Notes

Something noteworthy that comes from our normalization algorithm is that we
never directly subtract the floor curve from the raw mdct output. If you look at
early versions of the reference vorbis encoder (or psytune.c in the most recent
version) you notice that there is a lookup table for inverse energy of the floor
values which is multiplied against the raw mdct output. This naturally serves as
a more efficient method of logarithmic subtraction than real division. This is
an obvious approach for the subtraction of the floor curve.

The normalization and coupling algorithms actually keep track of the quantized
energy for each point. Avoids the problem of subtraction altogether! This
approach is not necessarily more efficient than manual division, but it
simplifies the logic during residue encoding. This is actually quite a clever
approach where we both expose more useful information to the encoder and
partially offset the resulting complexity.

This was initially a bit of a trip-up for me When adapting these methods for
FFmpeg's encoder. I think it would be productive to find a way to better
document this behavior. A major challenge in implementing a Vorbis encoder is
dealing with the fact that Vorbis as a codec is defined via the decoder.

*FFmpeg is an open source software. Being such, if you'd like to see the actual
implementation of this, feel free to reference commits: SOURCE, SOURCE, SOURCE.*

*Have questions, comments, or suggestions? As usual, please feel free to contact
me via email or social media above.*
