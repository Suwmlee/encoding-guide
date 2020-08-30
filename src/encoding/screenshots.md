# Taking Screenshots

Taking simple screenshots in VapourSynth is very easy.
If you're using a previewer, you can likely use that instead, but it might still be useful to know how to take screenshots via VapourSynth directly.

We recommend using `awsmfunc.ScreenGen`.
This has two advantages:

1. You save frame numbers and can easily reference these again, e.g. if you want to redo your screenshots.
2. It takes care of proper conversion and compression for you, which might not be the case with some previewers (e.g. VSEdit).

To use `ScreenGen`, create a new folder you want your screenshots in, e.g. "Screenshots", and a file with the frame numbers you'd like to screenshot, e.g.

```
26765
76960
82945
92742
127245
```

Then, at the bottom of your VapourSynth script, put

```
awf.ScreenGen(src, "Screenshots", "a")
```

`a` is what is put after the frame number.
This is useful for staying organized and sorting screenshots, as well as preventing unnecessary overwriting of screenshots.

Now, run your script in the command line (or reload in a previewer):

```sh
python vapoursynth_script.vpy
```

Done!
Your screenshots should now be in the given folder.

# Comparing Source vs. Encode

Comparing the source against your encode allows potential downloaders to judge the quality of your encode easily.
When taking these, it is important to include the frame types you are comparing, as e.g. comparing two `I` frames will lead to extremely favorable results.
You can do this using `awsmfunc.FrameInfo`:

```py
src = awf.FrameInfo(src, "Source")
encode = awf.FrameInfo(encode, "Encode")
```

If you'd like to compare these in your previewer, it's recommended to interleave them:

```py
out = core.std.Interleave([src, encode])
```

However, if you're taking your screenshots with `ScreenGen`, it's easier not to do that and just run two `ScreenGen` calls:

```py
src = awf.FrameInfo(src, "Source")
awf.ScreenGen(src, "Screenshots", "a")
encode = awf.FrameInfo(encode, "Encode")
awf.ScreenGen(encode, "Screenshots", "b")
```

Note that `"a"` was substituted for `"b"` in the encode's `ScreenGen`.
This will allow you to sort your folder by name and have every source screenshot followed by an encode screenshot, making uploading easier.

### HDR comparisons

For comparing an HDR source to an HDR encode, it's recommended to tonemap.
This process is destructive, but you should still be able to tell what's warped, smoothed etc.

The recommended function for this is `awsmfunc.DynamicTonemap`:

```py
src = awf.DynamicTonemap(src, src_fmt=False, libplacebo=False)
encode = awf.DynamicTonemap(encode, src_fmt=False, libplacebo=False)
```

Note that we've disabled `src_fmt` and `libplacebo` here.
Setting the former to `True` will output 10-bit 4:2:0, which is suboptimal, as screenshots are usually presented in 8-bit RGB (no chroma subsampling).
The latter is recommended for comparisons because using libplacebo makes your tonemaps more likely to differ in brightness, making comparing more difficult.

## Choosing frames

When taking screenshots, it is important to not make your encode look deceptively transparent.
To do so, you need to make sure you're screenshotting the proper frame types as well as content-wise differing kinds of frames.

Luckily, there's not a lot to remember here:

* Your encode's screenshots should *always* be *B* type frames.
* Your source's screenshots should *never* be *I* type frames.
* Your comparisons should include dark scenes, bright scenes, close-up shots, long-range shots, static scenes, high action scenes, and whatever you have in-between.

# Comparing Different Sources

When comparing different sources, you should proceed similarly to comparing source vs. encode.
However, you'll likely encounter differing crops, resolutions or tints, all of which get in the way of comparing.

For differing crops, simple add borders back:

```py
src_b = src_b.std.AddBorders(left=X, right=Y, top=Z, bottom=A)
```

For differing resolutions, it's recommended to use a simple spline resize:

```py
src_b = src_b.resize.Spline36(src_a.width, src_a.height, dither_type="error_diffusion")
```

If one source is HDR and the other one is SDR, you can use `awsmfunc.DynamicTonemap`:

```py
src_b = awf.DynamicTonemap(src_b, src_fmt=False)
```

For different tints, refer to the [tinting chapter](../filtering/detinting.md).
