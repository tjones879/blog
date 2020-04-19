---
title: "Building a Drone -- IMU"
date: 2019-07-06T00:00:00-06:00
draft: "true"
---

<!--more-->

Simulation of an IMU
========================

We have a few sources of error when using an inertial measurement unit.

Gyroscopic bias -- This is the average accumulated drift due to the gyroscope. Generally measured in deg/hour




White Noise
=========================



Bias
========================

We are using the Wiener process to model brownian motion for long-term
measurement bias.

{{<highlight py "linenos=inline">}}

def brownian(x0, n, dt, delta, out=None):
    """
    Generate brownian motion:
        X(t) = X(0) + N(0, delta**2 * t; 0, t)

    where N(a, b; t0, t1) is a normally distributed random
    variable with mean `a` and variance `b`. The 
    """
    x0 = np.asarray(x0)
    r = norm.rvs(size=x0.shape + (n,),
                 scale=delta*sqrt(dt))

    if out is None:
        out = np.empty(r.shape)

    np.cumsum(r, axis=-1, out=out)
    out += np.expand_dims(x0, axis=-1)

{{</highlight>}}






Allan Variance
========================




Resources
========================

https://github.com/ethz-asl/kalibr/wiki/IMU-Noise-Model





Random content

asdad 
asdasdf

{{< highlight c "linenos=inline">}}
int var = 1;
typedef struct clazz {
	int a;
	int b;
} clazz;
{{< /highlight >}}
{{< highlight c "linenos=inline">}}
int ff_psy_vorbis_block_frame(VorbisPsyContext *vpctx, float *audio,
                              int ch, int frame_size, int block_size)
{
    int i, block_flag = 1;
    int blocks = frame_size / block_size;
    float last_var;
    const float eps = 0.0001f;
    float *var = vpctx->variance + ch * blocks;

    for (i = 0; i < frame_size; i++) {
        apply_filter(&vpctx->filter[0], audio[i], vpctx->filter_delay + 4 * ch);
        apply_filter(&vpctx->filter[1], audio[i], vpctx->filter_delay + 4 * ch + 2);
    }

    for (i = 0; i < blocks; i++) {
        last_var = var[i];
        var[i] = variance(audio + i * block_size, block_size, vpctx->fdsp);

        /* A small constant is added to the threshold in order to prevent false
         * transients from being detected when quiet sounds follow near-silence */
        if (var[i] > vpctx->preecho_thresh * last_var + eps)
            block_flag = 0;
    }

    return block_flag;
}
{{< /highlight >}}
