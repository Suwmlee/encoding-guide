# Black & White Clips: Working in GRAY

Because of the YUV format saving luma in a separate plane, working with black and white movies makes our lives a lot easier when filtering, as we can extract the luma plane and work solely on that:

```py
y = src.std.ShufflePlanes(0, vs.GRAY)
```

The `get_y` function in `vsutil` does the same thing.
With our `y` clip, we can perform functions without mod2 limitations.
For example, we can perform odd crops:

```py
crop = y.std.Crop(left=10)
```

Additionally, as filters are only applied on one plane, this can speed up our script.
We also don't have to worry about filters like `f3kdb` altering our chroma planes in unwanted ways, such as graining them.

However, when we're done with our clip, we usually want to export the clip as YUV.
There are two options here:

### 1. Using fake chroma

Using fake chroma is quick and easy and has the advantage that any accidental chroma offsets in the source (e.g. chroma grain) will be removed.
All this requires is constant chroma (meaning no tint changes) and mod2 luma.

The simplest option is for `u = v = 128` (8-bit):

```py
out = y.resize.Point(format=vs.YUV420P8)
```

If you have an uneven luma, just pad it with `awsmfunc.fb`.
Assuming you want to pad the left:

```py
y = core.std.StackHorizontal([y.std.BlankClip(width=1), y])
y = awf.fb(y, left=1)
out = y.resize.Point(format=vs.YUV420P8)
```

Alternatively, if your source's chroma isn't a neutral gray, use `std.BlankClip`:

```py
blank = y.std.BlankClip(color=[0, 132, 124])
out = core.std.ShufflePlanes([y, blank], [0, 1, 2], vs.YUV)
```

### 2. Using original chroma (resized if necessary).

This has the advantage that, if there is actual important chroma information (e.g. slight sepia tints), this will be preserved.
Just use `ShufflePlanes` on your clips:

```py
out = core.std.ShufflePlanes([y, src], [0, 1, 2], vs.YUV)
```

However, if you've resized or cropped, this becomes a bit more difficult.
You might have to shift or resize the chroma appropriately (see [the chroma resampling chapter](../filtering/chroma_rs.md) for explanations).

If you've cropped, extract and shift accordingly. We will use `split` and `join` from `vsutil` to extract and merge planes:

```py
y, u, v = split(src)
crop_left = 1
y = y.std.Crop(left=crop_left)
u = u.resize.Spline36(src_left=crop_left / 2)
v = v.resize.Spline36(src_left=crop_left / 2)
out = join([y, u, v])
```

If you've resized, you need to shift and resize chroma:

```py
y, u, v = split(src)
w, h = 1280, 720
y = y.resize.Spline36(w, h)
u = u.resize.Spline36(w / 2, h / 2, src_left=.25 - .25 * src.width / w)
v = v.resize.Spline36(w / 2, h / 2, src_left=.25 - .25 * src.width / w)
out = join([y, u, v])
```

Combining cropping and shifting, whereby we pad the crop and use `awsmfunc.fb` to create a fake line:

```py
y, u, v = split(src)
w, h = 1280, 720
crop_left, crop_bottom = 1, 1

y = y.std.Crop(left=crop_left, bottom=crop_bottom)
y = y.resize.Spline36(w - 1, h - 1)
y = core.std.StackHorizontal([y.std.BlankClip(width=1), y])
y = awf.fb(left=1)

u = u.resize.Spline36(w / 2, h / 2, src_left=crop_left / 2 + (.25 - .25 * src.width / w), src_height=u.height - crop_bottom / 2)

v = v.resize.Spline36(w / 2, h / 2, src_left=crop_left / 2 + (.25 - .25 * src.width / w), src_height=u.height - crop_bottom / 2)

out = join([y, u, v])
```

If you don't understand exactly what's going on here and you encounter a situation like this, ask someone more experienced for help.

