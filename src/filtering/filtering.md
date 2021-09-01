While working on a video,
one will usually encounter some visual artifacts,
such as banding, darkened borders, etc.
As these are not visually pleasant,
and for the most part aren't intended by the original creator,
it's prefered to use video filters to fix them.

## Scene-filtering

As every filter is destructive in some way,
it is desirable to only apply them whenever necessary.
This is usually done using `ReplaceFramesSimple`,
which is in the [`RemapFrames`](https://github.com/Irrational-Encoding-Wizardry/Vapoursynth-RemapFrames) plugin.
`RemapFramesSimple` can also be called with `Rfs`.
Alternatively, one can use a Python solution, e.g. std.Trim and addition.
However, `RemapFrames` tends to be faster,
especially for larger sets of replacement mappings.

Let's look at an example of applying the [`f3kdb`](filtering/debanding##neo_f3kdb) debanding filter to frames 100 through 200 and 500 through 750:

```py
src = core.ffms2.Source("video.mkv")
deband = source.neo_f3kdb.Deband(src)

replaced = core.remap.Rfs(src, deband, mappings="[100 200] [500 750]")
```

There are various wrappers around both the plugin and the Python method, notably [`awsmfunc.rfs`](https://git.concertos.live/AHD/awsmfunc) for the former and [`lvsfunc.util.replace_frames`](https://lvsfunc.encode.moe/en/latest/#lvsfunc.util.replace_ranges) for the latter.

## Filter order

In order for filters to work correctly and not be counterproductive,
it is important to apply them in the proper order.
This is especially important for filters like debanders and grainers,
as putting these before a resize can completely negate their effect.

A generally acceptable order would be:
1. Load the source
2. Crop
3. Raise bit depth
4. Detint
5. Fix dirty lines
6. Deblock
7. Resize
8. Denoise
9. Anti-aliasing
10. Dering (dehalo)
11. Deband
12. Grain
13. Dither to output bit depth

Keep in mind that this is just a general recommendation.
There can always be a case where you might want to deviate,
e.g. if you're using a fast denoiser like KNLMeansCL,
you can do this before resizing.
