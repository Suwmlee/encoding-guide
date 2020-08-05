While most movies are produced at 2K resolution and most anime are made
at 720p, Blu-rays are almost always 1080p and UHD Blu-rays are all 4K.
This means the mastering house often has to upscale footage. These
upscales are pretty much always terrible, but luckily, some are
reversible. Since anime is usually released at a higher resolution than
the source images, and bilinear or bicubic upscales are very common,
most descalers are written for anime, and it's the main place where
you'll need to descale. Live action content usually can't be descaled
because of bad proprietary scalers (often QTEC or the likes), hence most
live action encoders don't know or consider descaling.

So, if you're encoding anime, always make sure to check what the source
images are. You can use <https://anibin.blogspot.com/> for this, run
screenshots through `getnative`[^36], or simply try it out yourself. The
last option is obviously the best way to go about this, but getnative is
usually very good, too, and is a lot easier. Anibin, while also useful,
won't always get you the correct resolution.

In order to perform a descale, you should be using `fvsfunc`:

    import fvsfunc as fvf

    descaled = fvf.Debilinear(src, 1280, 720, yuv444=False)

In the above example, we're descaling a bilinear upscale to 720p and
downscaling the chroma with `Spline36` to 360p. If you're encoding anime
for a site/group that doesn't care about hardware compatibility, you'll
probably want to turn on `yuv444` and change your encode settings
accordingly.

`Descale` supports bilinear, bicubic, and spline upscale kernels. Each
of these, apart from `Debilinear`, also has its own parameters. For
`Debicubic`, these are:

-   `b`: between 0 and 1, this is equivalent to the blur applied.

-   `c`: also between 0 and 1, this is the sharpening applied.

The most common cases are `b=1/3` and `c=1/3`, which are the default
values, `b=0` and `c=1`, which is oversharpened bicubic, and `b=1` and
`c=0`, which is blurred bicubic. In between values are quite common,
too, however.

Similarly, `Delanczos` has the `taps` option, and spline upscales can be
reversed for `Spline36` upscales with `Despline36` and `Despline16` for
`Spline16` upscales.

Once you've descaled, you might want to upscale back to the 1080p or
2160p. If you don't have a GPU available, you can do this via `nnedi3`
or more specifically, `edi3_rpow2` or `nnedi3_rpow2`:

    from edi3_rpow2 import nnedi3_rpow2

    descaled = fvf.Debilinear(src, 1280, 720)
    upscaled = nnedi3_rpow2(descaled, 2).resize.Spline36(1920, 1080)
    out = core.std.Merge(upscaled, src, [0, 1])

What we're doing here is descaling a bilinear upscale to 720p, then
using `nnedi3` to upscale it to 1440p, downscaling that back to 1080p,
then merging it with the source's chroma. Those with a GPU available
should refer to the `FSRCNNX` example in the resizing
section.[3.1.1](#resize){reference-type="ref" reference="resize"}\
There are multiple reasons you might want to do this:

-   Most people don't have a video player set up to properly upscale the
    footage.

-   Those who aren't very informed often think higher resolution =
    better quality, hence 1080p is more popular.

-   Lots of private trackers only allow 720p and 1080p footage. Maybe
    you don't want to hurt the chroma or the original resolution is in
    between (810p and 900p are very common) and you want to upscale to
    1080p instead of downscaling to 720p.

Another important thing to note is that credits and other text is often
added after upscaling, hence you need to use a mask to not ruin these.
Luckily, you can simply add an `M` after the descale name
(`DebilinearM`) and you'll get a mask applied. However, this
significantly slows down the descale process, so you may want to
scenefilter here.

On top of the aforementioned common descale methods, there are a few
more filters worth considering, although they all do practically the
same thing, which is downscaling line art (aka edges) and rescaling them
to the source resolution. This is especially useful if lots of dither
was added after upscaling.

-   `DescaleAA`: part of `fvsfunc`, uses a `Prewitt` mask to find line
    art and rescales that.

-   `InsaneAA`: Uses a strengthened `Sobel` mask and a mixture of both
    `eedi3` and `nnedi3`.

Personally, I don't like upscaling it back and would just stick with a
YUV444 encode. If you'd like to do this, however, you can also consider
trying to write your own mask. An example would be (going off the
previous code):

    mask = kgf.retinex_edgemask(src).std.Binarize(15000).std.Inflate()
    new_y = core.std.MaskedMerge(src, upscaled, mask)
    new_clip = core.std.ShufflePlanes([new_y, u, v], [0, 0, 0], vs.YUV)

Beware with descaling, however, that using incorrect resolutions or
kernels will only intensify issues brought about by bad upscaling, such
as ringing and aliasing. It's for this reason that you need to be sure
you're using the correct descale, so always manually double-check your
descale. You can do this via something like the following code:

    import fvsfunc as fvf
    from vsutil import join, split
    y, u, v = split(source) # this splits the source planes into separate clips
    descale = fvf.Debilinear(y, 1280, 720)
    rescale = descale.resize.Bilinear(1920, 1080)
    merge = join([y, u, v])
    out = core.std.Interleave([source, merge])

If you can't determine the correct kernel and resolution, just downscale
with a normal `Spline36` resize. It's usually easiest to determine
source resolution and kernels in brighter frames with lots of blur-free
details.

In order to illustrate the difference, here are examples of rescaled
footage. Do note that YUV444 downscales scaled back up by a video player
will look better.

![Source Blu-ray with 720p footage upscaled to 1080p via a bicubic
filter on the left, rescale with `Debicubic` and `nnedi3` on the
right.](Pictures/bicubic.png){#fig:6}

It's important to note that this is certainly possible with live action
footage as well. An example would be the Game of Thrones season 1 UHD
Blu-rays, which are bilinear upscales. While not as noticeable in
screenshots, the difference is stunning during playback.

![Source UHD Blu-ray with 1080p footage upscaled to 2160p via a bilinear
filter on the left, rescale with `Debilinear` and `nnedi3` on the
right.](Pictures/bilinear_before2.png){#fig:7}

![Source UHD Blu-ray with 1080p footage upscaled to 2160p via a bilinear
filter on the left, rescale with `Debilinear` and `nnedi3` on the
right.](Pictures/bilinear_after2.png){#fig:7}

If your video seems to have multiple source resolutions in every frame
(i.. different layers are in different resolutions), which you can
notice by `getnative` outputting multiple results, your best bet is to
downscale to the lowest resolution via `Spline36`. While you technically
can mask each layer to descale all of them to their source resolution,
then scale each one back up, this is far too much effort for it to be
worth it.
