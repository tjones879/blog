---
title: "Building a Drone -- ESC"
date: 2017-08-07T17:38:05-06:00
draft: "true"
---
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
<!--more-->
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
