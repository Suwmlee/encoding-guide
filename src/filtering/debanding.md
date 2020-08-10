# Debanding

This is the most common issue one will encounter. Banding usually
happens when bitstarving and poor settings lead to smoother gradients
becoming abrupt color changes, which obviously ends up looking bad. The
good news is higher bit depths can help with this issue, since more
values are available to create the gradients. Because of this, lots of
debanding is done in 16 bit, then dithered down to 10 or 8 bit again
after the filter process is done.\
One important thing to note about debanding is that you should try to
always use a mask with it, e.. an edge mask or similar. See
[the masking section](masking) for
details!\
There are three great tools for VapourSynth that are used to fix
banding: [`neo_f3kdb`](https://github.com/HomeOfAviSynthPlusEvolution/neo_f3kdb/), `fvsfunc`'s `gradfun3`, which has a built-in
mask, and `vs-placebo`'s `placebo.Deband`.\
Let's take a look at `neo_f3kdb` first: The default relevant code for
VapourSynth looks as follows:

```py
deband = core.neo_f3kdb.deband(src=clip, range=15, y=64, cb=64, cr=64, grainy=64, grainc=64, dynamic_grain=False, output_depth=16, sample_mode=2)
```

These settings may come off as self-explanatory for some, but here's
what they do:

-   `src` This is obviously your source clip.

-   `range` This specifies the range of pixels that are used to
    calculate whether something is banded. A higher range means more
    pixels are used for calculation, meaning it requires more processing
    power. The default of 15 should usually be fine.

-   `y` The most important setting, since most (noticeable) banding
    takes place on the luma plane. It specifies how big the difference
    has to be for something on the luma plane to be considered as
    banded. You should start low and slowly but surely build this up
    until the banding is gone. If it's set too high, lots of details
    will be seen as banding and hence be blurred.

-   `cb` and `cr` The same as `y` but for chroma. However, banding on
    the chroma planes is quite uncommon, so you can often leave this
    off.

-   `grainy` and `grainc` In order to keep banding from re-occurring and
    to counteract smoothing, grain is usually added after the debanding
    process. However, as this fake grain is quite noticeable, it's
    recommended to be conservative. Alternatively, you can use a custom
    grainer, which will get you a far nicer output (see
    [3.2.10](#graining){reference-type="ref" reference="graining"}).

-   `dynamic_grain` By default, grain added by `f3kdb` is static. This
    compresses better, since there's obviously less variation, but it
    usually looks off with live action content, so it's normally
    recommended to set this to `True` unless you're working with
    animated content.

-   `sample_mode` Is explained in the README.  Consider switching to 4, since it might have less detail loss.

-   `output_depth` You should set this to whatever the bit depth you
    want to work in after debanding is. If you're working in 8 bit the
    entire time, you can just leave out this option.

Here's an example of very simple debanding:

![Source on left,
`deband = core.f3kdb.Deband(src, y=64, cr=0, cb=0, grainy=32, grainc=0, range=15, keep_tv_range=True)`
on right. Zoomed in with a factor of
2.](Pictures/banding_before3.png){#fig:1}

![Source on left,
`deband = core.f3kdb.Deband(src, y=64, cr=0, cb=0, grainy=32, grainc=0, range=15, keep_tv_range=True)`
on right. Zoomed in with a factor of
2.](Pictures/banding_after3.png){#fig:1}

It's recommended to use a mask along with this for debanding, specifically
`retinex_edgemask` or `debandmask`
(albeit the former is a lot slower).  More on this in [the masking section](masking)\
\
The most commonly used alternative is `gradfun3`, which is likely mainly
less popular due to its less straightforward parameters. It works by
smoothing the source via a [bilateral filter](https://en.wikipedia.org/wiki/Bilateral_filter), limiting this via
`mvf.LimitFilter` to the values specified by `thr` and `elast`, then
merging with the source via its internal mask (although using an
external mask is also possible). While it's possible to use it for
descaling, most prefer to do so in another step.\
Many people believe `gradfun3` produces smoother results than `f3kdb`.
As it has more options than `f3kdb` and one doesn't have to bother with
masks most of the time when using it, it's certainly worth knowing how
to use:

    import fvsfunc as fvf
    deband = fvf.GradFun3(src, thr=0.35, radius=12, elast=3.0, mask=2, mode=3, ampo=1, ampn=0, pat=32, dyn=False, staticnoise=False, smode=2, thr_det=2 + round(max(thr - 0.35, 0) / 0.3), debug=False, thrc=thr, radiusc=radius, elastc=elast, planes=list(range(src.format.num_planes)), ref=src, bits=src.format.bits_per_sample) # + resizing variables

Lots of these values are for `fmtconv` bit depth conversion, so its
[documentation](https://github.com/EleonoreMizo/fmtconv/blob/master/doc/fmtconv.html) can prove to be helpful. Descaling in `GradFun3`
isn't much different from other descalers, so I won't discuss this. Some
of the other values that might be of interest are:

-   `thr` is equivalent to `y`, `cb`, and `cr` in what it does. You'll
    likely want to raise or lower it.

-   `radius` has the same effect as `f3kdb`'s `range`.

-   `smode` sets the smooth mode. It's usually best left at its default,
    a bilateral filter[^19], or set to 5 if you'd like to use a
    CUDA-enabled GPU instead of your CPU. Uses `ref` (defaults to input
    clip) as a reference clip.

-   `mask` disables the mask if set to 0. Otherwise, it sets the amount
    of `std.Maximum` and `std.Minimum` calls to be made.

-   `planes` sets which planes should be processed.

-   `mode` is the dither mode used by `fmtconv`.

-   `ampn` and `staticnoise` set how much noise should be added by
    `fmtconv` and whether it should be static. Worth tinkering with for
    live action content. Note that these only do anything if you feed
    `GradFun3` a sub-16 bit depth clip.

-   `debug` allows you to view the mask.

-   `elast` is \"the elasticity of the soft threshold.\" Higher values
    will do more blending between the debanded and the source clip.

For a more in-depth explanation of what `thr` and `elast` do, check the
algorithm explanation in [`mvsfunc`](https://github.com/HomeOfVapourSynthEvolution/mvsfunc/blob/master/mvsfunc.py#L1735).

The third debander worth considering is `placebo.Deband`. It uses the
mpv debander, which just averages pixels within a range and outputs the
average if the difference is below a threshold. The algorithm is
explained in the [source code](https://github.com/haasn/libplacebo/blob/master/src/shaders/sampling.c#L167). As this function is fairly new and
still has its quirks, it's best to refer to the [README](https://github.com/Lypheo/vs-placebo#placebodebandclip-clip-int-planes--1-int-iterations--1-float-threshold--40-float-radius--160-float-grain--60-int-dither--true-int-dither_algo--0) and ask
experience encoders on IRC for advice.

If your debanded clip had very little grain compared to parts with no
banding, you should consider using a separate function to add matched
grain so the scenes blend together easier. If there was lots of grain,
you might want to consider `adptvgrnMod`, `adaptive_grain` or
`GrainFactory3`; for less obvious grain or simply for brighter scenes
where there'd usually be very little grain, you can also use
`grain.Add`. This topic will be further elaborated later in
[the graining section](graining).\
Here's an example from Mirai (full script later):

![Source on left, filtered on right. Banding might seem hard to spot
here, but I can't include larger images in this PDF. The idea should be
obvious, though.](Pictures/banding_graining.png){#fig:3}

If you want to automate your banding detection, you can use a detection
function based on `bandmask`[^23] called `banddtct`[^24]. Make sure to
adjust the values properly and check the full output. A forum post
explaining it is linked in [the posts section](posts). You can also just run `adptvgrnMod` or
`adaptive_grain` with a high `luma_scaling` value in hopes that the
grain covers it up fully. More on this in
[the graining section](graining).

# Deblocking

Deblocking is mostly equivalent to smoothing the source, usually with
another mask on top. The most popular function here is `Deblock_QED`
from `havsfunc`. The main parameters are

-   `quant1`: Strength of block edge deblocking. Default is 24. You may
    want to raise this value significantly.

-   `quant2`: Strength of block internal deblocking. Default is 26.
    Again, raising this value may prove to be beneficial.

Other popular options are `deblock.Deblock`, which is quite strong, but
almost always works,\
`dfttest.DFTTest`, which is weaker, but still quite aggressive, and
`fvf.AutoDeblock`, which is quite useful for deblocking MPEG-2 sources
and can be applied on the entire video. Another popular method is to
simply deband, as deblocking and debanding are very similar. This is a
decent option for AVC Blu-ray sources.
