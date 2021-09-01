Cropping will almost always be necessary with live action content,
as black bars are almost always applied to fit a 16:9 frame.

If the to-be-cropped content has a resolution that is a multiple of 2 in each direction,
this is a very straightforward process:

```py
crop = src.std.Crop(top=2, bottom=2, left=2, right=2)
```

However, when this is not the case,
things get a little bit more complicated.
No matter what,
you must always crop away as much as you can before proceeding.

If you're working with a completely black-and-white film, read [the appendix entry on these](appendix/gray.html).
The following only applies to videos with colors.

Then, read through this guide's [`FillBorders`](dirty_lines.html#fillborders) explanation from the dirty lines subchapter.
This will explain what to do if you do not plan on resizing.

If you do plan on resizing,
proceed according to the [resizing note](dirty_lines.html#resizing) from the dirty lines subchapter.
