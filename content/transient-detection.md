---
title: "Transient Detection"
date: 2017-08-07T18:38:10-06:00
---
<!--more-->

*This is part of a series of posts designed to document the processes I followed
to improve FFmpeg's native vorbis encoder.*

In many modern lossy audio codecs, there occurs a phenomenon called pre-echo. It
can be easily noticed as hearing a faint echo of a sound before it should
actually occur. It actually sounds exactly like it is described.

[This](https://en.wikipedia.org/wiki/File:Genesis-Duke-LPpreecho.ogg) sample
found on Wikipedia is amplified to be more noticeable. It is possible to hear
an echo in the first couple seconds. Although it is not always noticeable in
practice, pre-echo is significant problem that needs solved in many codecs.

This challenge has some interesting qualities:

1. While we know why pre-echos occur, there are only heuristics that help
   predict when a pre-echo will possibly occur. Our attempts to prevent them
   down to estimation and must balance aggressiveness to not negatively impact
   the quality of audio in other ways.

2. Prevention of pre-echo must be done very efficiently. We hope to contribute
   little to the overall runtime of the encoder.

3. Different listeners may perceive the output of our system differently than
   others. With different listening equipment and the wide variety of audio
   files available, we must not be too aggressive in transient avoidance.

Our requirements fall partially at odds with each other, so we must be careful
in balancing each component.

### Categorization of Sound

If you recall from high-school physics, sound waves follow a periodic nature.
This obviously changes based on the source of our sound. We can definitively say
that a normal cello's A string will produce a sound at a primary frequency of
220 Hz.

In contrast, a *transient signal* can be seen as a short burst signal that
doesn't show the same periodic behavior of other audio[^2]. Common sources
include drum hits and cymbals. It is common for transient signals occur at high
frequencies, and the transient effect often exists at a frequency different
vastly different from the bulk of the signal.

/* Imagine that we want to describe a series of whistles which can produce pure
tones. We can either explain it as a series of sounds at certain time intervals,
or we can understand it as a series of notes at certain frequencies. This song
can be translated between the *time domain* and *frequency domain*. It turns out
that encoding audio in the frequency domain is very efficient. */

Most of our modern audio codecs use transforms[^transforms] to convert audio
samples from the time domain to the frequency domain by approximating our data
with a series of periodic functions. The purpose is to be able to estimate the
signal at any point in time just from these functions. Vorbis I uses the
MDCT-IV.

One of our goals is to manipulate the uncertainty principle in a way that
improves quality. When we partition samples into larger blocks, we have a
greater resolution in the __frequency domain__. We can more accurately analyze
the frequency components of our signals. However, we lose our certainty in
saying when those signals occur. In other words, we get a poorer resolution in
the __time domain__. Intuitively, the opposite occurs when we lower the block
size.

For efficiency reasons, encoders typically use large blocks of samples. This
means that we get great spectral resolution, but we lose information about when
a signal occurs. This signal essentially gets spread over the entire block in
time.

Much of what we hear in music *can* be approximated fairly well.  Transient
signals are an exception to this and are very difficult to approximate in the
amount of terms available within our frequency transform. This failure causes
the decoder to decode signals before or after they really occur.

### What is a possible solution?

Many audio formats, Vorbis included, allow for variable block lengths. Long
blocks are far more efficient to transform and encode, so we use them as
default. The obvious solution is to switch to shorter blocks when its necessary!
It's really as simple as that, and short blocks will contribute a noticeable
improvement in quality.

Our frequency transform will always introduce uncertainty in time locality, so
we must find out how largest blocks we are allowed to use before we can perceive
them. Psychoacoustics deal with understanding how the human auditory system
actually works. Well-known researchers found in the mid 20th century that
special temporal masking occurs before and after loud noise. Noise is partially
masked for 2.5ms before you actually perceive a sound.

Some quick napkin math reveals:

$$2.5ms * 1s / 1000ms = X samples * 1s / 44000 samples$$
$$X = 2.5 * 44 = 110 samples$$

Our closest power of 2 is 128 samples. This means that if use 128-sample blocks,
then there is a very high chance of masking the pre-echo with the signal itself!

### How do we detect transients?

Now the story is set: we (1) know the challenges the system imposes, (2)
understand the behavior of transient signals, and (3) know how to limit their
impact. The only thing that is left is to be able to accurately determine when
this is occurring.

In practice, pre-echos are not as obvious as the sample above. Looking at the
raw data will not reveal much of anything by itself.

First remember that transient signals will often emit high frequencies that do
not necessarily align with their fundamental frequency. Let's filter out low
frequency signals.

I arbitrarily chose 1/4 of the nyquist frequency as our critical frequency and
used a 4th order highpass butterworth filter. This will introduce minimal
distortion in area of interest. Since the filtered data will never be encoded,
we do not worry about the poor impulse response from the IIR. Our filters are
also cheap enough computationally and stable numerically to avoid any extra
scrutiny.

We also know that low-frequency signals often dominate the appearance of the
total waveform, but typically pose little difficulty for quantization.

We now have a more useful set of data, so how do we classify whether a signal is
transient? The most common sources of transients include sharp attacks (light
brass and percussion). You could monitor the input stream and just wait for any
sudden movements. Some encoders use a method similar to this by breaking down
frames into bands and summing the total energy for each. This works well for
isolated attacks but becomes less effective with busy tracks (much of modern pop
music).

You could possibly estimate the derivative of the input
stream and wait for a sharp jump. However, that is inefficient and too prone to
error.

the input.  We know that low-frequency sources are not likely to pose
quantization problems, and they often dominate the appearance of the overall
waveform.

filtered samples

Our data is more easily analysed, and only the part we are interested in is
presented to us. We have discussed that transients are often non-periodic, and
as a result, we can also suspect that the block containing it will have a
significantly larger variance than previous samples.

If we multiply previous blocks by some threshold and find that our current block
is larger, than we can predict, with some amount of certainty, that a transient
has occurred and its pre-echo would be significant.

We must avoid switching transforms too aggressively to avoid noticeable
artifacts.

### Other Resources

If the signal processing components interest you, I'd highly recommend the
wonderful [everyday dsp](). This will go all the way to basic filtering in a
clear and visually engaging way. Want to go more in-depth?

Want to know more about audio encoding? The hydrogen media [forums]() and
[wiki]() are great at describing some of these algorithms from a high level.
Some of FFmpeg's native encoders provide very readable implementations if you
want a deeper understanding.

*FFmpeg is an open source software. Being such, if you'd like to see the actual
implementation of this, feel free to reference the source in a git repo near
you!*

*Have questions, comments, or suggestions? As usual, please feel free to contact
me via !!!email!!! or !!!social media!!!.*

[^preecho]: a.
[^transforms]: a.
