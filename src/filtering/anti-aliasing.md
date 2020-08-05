This is likely the most commonly known issue. If you want to fix this,
first make sure the issue stems from actual aliasing, not poor
upscaling. If you've done this, the tool I'd recommend is the `TAAmbk`
suite[^33]:

    import vsTAAmbk as taa

    aa = taa.TAAmbk(clip, aatype=1, aatypeu=None, aatypev=None, preaa=0, strength=0.0, cycle=0,
                            mtype=None, mclip=None, mthr=None, mthr2=None, mlthresh=None, 
                            mpand=(1, 0), txtmask=0, txtfade=0, thin=0, dark=0.0, sharp=0, aarepair=0, 
                            postaa=None, src=None, stabilize=0, down8=True, showmask=0, opencl=False, 
                            opencl_device=0, **args)

The GitHub README is quite extensive, but some additional comments are
necessary:

-   `aatype`: (Default: 1)\
    The value here can either be a number to indicate the AA type for
    the luma plane, or it can be a string to do the same.

                0: lambda clip, *args, **kwargs: type('', (), {'out': lambda: clip}),
                1: AAEedi2,
                2: AAEedi3,
                3: AANnedi3,
                4: AANnedi3UpscaleSangNom,
                5: AASpline64NRSangNom,
                6: AASpline64SangNom,
                -1: AAEedi2SangNom,
                -2: AAEedi3SangNom,
                -3: AANnedi3SangNom,
                'Eedi2': AAEedi2,
                'Eedi3': AAEedi3,
                'Nnedi3': AANnedi3,
                'Nnedi3UpscaleSangNom': AANnedi3UpscaleSangNom,
                'Spline64NrSangNom': AASpline64NRSangNom,
                'Spline64SangNom': AASpline64SangNom,
                'Eedi2SangNom': AAEedi2SangNom,
                'Eedi3SangNom': AAEedi3SangNom,
                'Nnedi3SangNom': AANnedi3SangNom,
                  'PointSangNom': AAPointSangNom,

    The ones I would suggest are `Eedi3`, `Nnedi3`, `Spline64SangNom`,
    and `Nnedi3SangNom`. Both of the `SangNom` modes are incredibly
    destructive and should only be used if absolutely necessary.
    `Nnedi3` is usually your best option; it's not very strong or
    destructive, but often good enough, and is fairly fast. `Eedi3` is
    unbelievably slow, but stronger than `Nnedi3` and not as destructive
    as the `SangNom` modes.

-   `aatypeu`: (Default: same as `aatype`)\
    Select main AA kernel for U plane when clip's format is YUV.

-   `aatypev`: (Default: same as `aatype`)\
    Select main AA kernel for V plane when clip's format is YUV.

-   `strength`: (Default: 0)\
    The strength of predown. Valid range is $[0, 0.5]$ Before applying
    main AA kernel, the clip will be downscaled to
    ($1-$strength)$\times$clip\_resolution first and then be upscaled to
    original resolution by main AA kernel. This may be benefit for clip
    which has terrible aliasing commonly caused by poor upscaling.
    Automatically disabled when using an AA kernel which is not suitable
    for upscaling. If possible, do not raise this, and *never* lower it.

-   `preaa`: (Default: 0)\
    Select the preaa mode.

    -   0: No preaa

    -   1: Vertical

    -   2: Horizontal

    -   -1: Both

    Perform a `preaa` before applying main AA kernel. `Preaa` is
    basically a simplified version of `daa`. Pretty useful for dealing
    with residual comb caused by poor deinterlacing. Otherwise, don't
    use it.

-   `cycle`: (Default: 0)\
    Set times of loop of main AA kernel. Use for very very terrible
    aliasing and 3D aliasing.

-   `mtype`: (Default: 1)\
    Select type of edge mask to be used. Currently there are three mask
    types:

    -   0: No mask

    -   1: `Canny` mask

    -   2: `Sobel` mask

    -   3: `Prewitt` mask

    Mask always be built under 8-bit scale. All of these options are
    fine, but you may want to test them and see what ends up looking the
    best.

-   `mclip`: (Default: None)\
    Use your own mask clip instead of building one. If `mclip` is set,
    script won't build another one. And you should take care of mask's
    resolution, bit-depth, format, etc by yourself.

-   `mthr`:\
    Size of the mask. The smaller value you give, the bigger mask you
    will get.

-   `mlthresh`: (Default None)\
    Set luma thresh for n-pass mask. Use a list or tuple to specify the
    sections of luma.

-   `mpand`: (Default: (1, 0))\
    Use a list or tuple to specify the loop of mask expanding and mask
    inpanding.

-   `txtmask`: (Default: 0)\
    Create a mask to protect white captions on screen. Value is the
    threshold of luma. Valid range is 0 255. When a area whose luma is
    greater than threshold and chroma is $128\pm2$, it will be
    considered as a caption.

-   `txtfade`: (Default: 0)\
    Set the length of fading. Useful for fading text.

-   `thin`: (Default: 0)\
    Warp the line by aWarpSharp2 before applying main AA kernel.

-   `dark`: (Default: 0.0)\
    Darken the line by Toon before applying main AA kernel.

-   `sharp`: (Default: 0)\
    Sharpen the clip after applying main AA kernel. \* 0: No sharpen. \*
    1 inf: LSFmod(defaults='old') \* 0 1: Simlar to Avisynth's
    sharpen() \* -1 0: LSFmod(defaults='fast') \* -1: Contra-Sharpen

    Whatever type of sharpen, larger absolute value of sharp means
    larger strength of sharpen.

-   `aarepair`: (Default: 0)\
    Use repair to remove artifacts introduced by main AA kernel.
    According to different repair mode, the pixel in src clip will be
    replaced by the median or average in 3x3 neighbour of processed
    clip. It's highly recommend to use repair when main AA kernel
    contain SangNom. For more information, check
    <http://www.vapoursynth.com/doc/plugins/rgvs.html#rgvs.Repair>. Hard
    to get it to work properly.

-   `postaa`: (Default: False)\
    Whether use soothe to counter the aliasing introduced by sharpening.

-   `src`: (Default: clip)\
    Use your own `src` clip for sharp, repair, mask merge, etc.

-   `stabilize`: (Default: 0)\
    Stabilize the temporal changes by `MVTools`. Value is the temporal
    radius. Valid range is $[0, 3]$.

-   `down8`: (Default: True)\
    If you set this to `True`, the clip will be down to 8-bit before
    applying main AA kernel and up it back to original bit-depth after
    applying main AA kernel. `LimitFilter` will be used to reduce the
    loss in depth conversion.

-   `showmask`: (Default: 0)\<br/\> Output the mask instead of processed
    clip if you set it to not 0. 0: Normal output; 1: Mask only; 2: tack
    mask and clip; 3: Interleave mask and clip; -1: Text mask only

-   `opencl`: (Default: False) Whether use opencl version of some
    plugins. Currently there are three plugins that can use opencl:

    -   TCannyCL

    -   EEDI3CL

    -   NNEDI3CL

    This may speed up the process, which is obviously great, since
    anti-aliasing is usually very slow.

-   `opencl_device`: (Default: 0)\
    Select an opencl device. To find out which one's the correct one, do

        core.nnedi3cl.NNEDI3CL(clip, 1, list_device=True).set_output()

-   `other parameters`:\
    Will be collected into a dict for particular aatype.

In most cases, `aatype=3` works well, and it's the method with the least
detail loss. Alternatively, `aatype=2` can also provide decent results.
What these two do is upscale the clip with `nnedi3` or the blurrier (and
slower) `eedi3` and downscale back, whereby the interpolation done often
fixes aliasing artifacts.

Note that there are a ton more very good anti-aliasing methods, as well
as many different mask types you can use (e.g. other edgemasks, clamping
one method's changes to those of another method etc.). However, most
methods are based on very similar methods to those `TAA` implements.

An interesting suite for anti-aliasing is the one found in
`lvsfunc`[^34]. It provides various functions that are very useful for
fixing strong aliasing that a normal `TAA` call might not handle very
well. The documentation for it[^35] is fantastic, so this guide won't go
deeper into its functions.

If your entire video suffers from aliasing, it's not all too unlikely
that you're dealing with a cheap upscale. In this case, descale or
resize first before deciding whether you need to perform any
anti-aliasing.

Here's an example of an anti-aliasing fix from Non Non Biyori -
Vacation:

![Source with aliasing on left, filtered on
right.](Pictures/aa.png){#fig:5}

In this example, the following was done:

    mask = kgf.retinex_edgemask(src).std.Binarize(65500).std.Maximum().std.Inflate()
    aa = taa.TAAmbk(src, aatype=2, mtype=0, opencl=True)
    out = core.std.MaskedMerge(src, aa, mask)
