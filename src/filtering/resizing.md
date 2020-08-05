Firstly, note that there'll be a separate section later on for
descaling. Here, I'll explain resizing and what resizers are good for
what.

If you want to resize, it's important to not alter the aspect ratio more
than necessary. If you're downscaling, first figure out what the width
and height should be. If you want to downscale to 720p, first crop, then
figure out whether you're scaling to 720 height or 1280 width. If the
former is the case, your width should be:

```py
width = round(720 / src.height / 2 * src.width) * 2
```

For the latter, the code to find your height is quite similar:

```py
height = round(1280 / src.width / 2 * src.height) * 2
```
You can also use the `cropresize` wrapper in [`awsmfunc`](https://git.concertos.live/AHD/awsmfunc/) to do these
calculations and resize. Using the `preset` parameter, this function
will automatically determine whether a width of 1280 or a height of 720
should be used based on the aspect ratio, and then perform the above
calculation on the reset.

```py
import awsmfunc as awf
resize = awf.cropresize(source, preset=720)
```

There are many resizers available. The most important ones are:

-   **Point** also known as nearest neighbor resizing, is the simplest
    resizer, as it doesn't really do anything other than enlargen each
    pixel or create the average of surrounding each pixel when
    downscaling. It produces awful results, but doesn't do any blurring
    when upscaling, hence it's very good for zooming in to check what
    each pixel's value is. It is also self-inverse, so you can scale up
    and then back down with it and get the same result you started with.

-   **Bilinear** resizing is very fast, but leads to very blurry results
    with noticeable aliasing.

-   **Bicubic** resizing is similarly fast, but also leads to quite
    blurry results with noticeable aliasing. You can modify parameters
    here for sharper results, but this will lead to even more aliasing.

-   **Lanczos** resizing is slower and gets you very sharp results.
    However, it creates very noticeable ringing artifacts.

-   **Blackmanminlobe** resizing, which you need to use `fmtconv`[^7]
    for, is a modified lanczos resizer with less ringing artifacts. This
    resizer is definitely worth considering for upscaling chroma for
    YUV444 encodes (more on this later).

-   **Spline** resizing is quite slow, but gets you very nice results.
    There are multiple spline resizers available, with `Spline16` being
    faster than `Spline36` with slightly worse results, and `Spline36`
    being similar enough to `Spline64` that there's no reason to use the
    latter. `Spline36` is the recommended resizer for downscaling
    content.

-   [**nnedi3**](https://gist.github.com/4re/342624c9e1a144a696c6) resizing is quite slow and can only upscale by powers
    of 2. It can then be combined with `Spline36` to scale down to the
    desired resolution. Results are significantly better than the
    aforementioned kernels.

-   [**FSRCNNX**](https://github.com/igv/FSRCNN-TensorFlow/releases) is a shader for mpv, which can be used via the
    [`vs-placebo`](https://github.com/Lypheo/vs-placebo) plugin. It provides far sharper results than
    `nnedi3`, but requires a GPU. It's recommended to use this for
    upscaling if you can. Like `nnedi3`, this is a doubler, so scale
    back down with spline36.

-   [**KrigBilateral**](https://gist.github.com/igv/a015fc885d5c22e6891820ad89555637) is another shader for mpv that you can use
    via `vs-placebo`. It's a very good chroma resizer. Again, a GPU is
    required.

While these screenshots should help you get a decent idea of the
differences between resizers, they're only small parts of single images.
If you want to get a better idea of how these resizers look, I recommend
doing these upscales yourself, watching them in motion, and interleaving
them (`std.Interleave`).

The difference between resizers when downscaling is a lot less
noticeable than when upscaling. However, it's not recommended to use
this as an excuse to be lazy with your choice of a resize kernel when
downscaling.

If you, say, want to [descale](descaling) a bilinear upscale from 1080p to 720p and scale
back to 1080p while maintaining source chroma, you'd do:

```py
import fvsfunc as fvf
from vsutil import get_y
descale = fvf.Debilinear(source, 1280, 720)
upscale = descale.placebo.Shader("FSRCNNX_x2_16-0-4-1.glsl", 1920, 1080)
y = get_y(upscale)
downscale = y.resize.Spline36(1920, 1080)
out = core.std.ShufflePlanes([downscale, source], [0, 1, 2], vs.YUV)
```

For significantly more in-depth explanations of different resizers, as
well as the topic of shifting, check
<https://guide.encode.moe/encoding/resampling.html>.

TL;DR: Use `core.resize.Spline36` for downscaling.
