Another very common issue, at least with live action content, is dirty
lines. These are usually found on the borders of video, where a row or
column of pixels exhibits usually a too low luma value compared to its
surrounding rows. Oftentimes, this is the due to improper downscaling,
more notably downscaling after applying borders. Dirty lines can also
occur because video editors often won't know that while they're working
in YUV422, meaning their height doesn't have to be mod2, consumer
products will be YUV420, meaning the height has to be mod2, leading to
extra black rows.\
Another form of dirty lines is exhibited when the chroma planes are
present on black bars. Usually, these should be cropped out. The
opposite can also occur, however, where the planes with legitimate luma
information lack chroma information.\
To illustrate what dirty lines might look like, here's an example of
`ContinuityFixer` and chroma-only `FillBorders`:\

![Source vs filtered of a dirty line fix from the D-Z0N3 encode of A
Silent Voice. Used `ContinuityFixer` on top three rows and `FillBorders`
on the two leftmost columns. Zoomed in with a factor of
15.](Pictures/dirt.png){#fig:4}

There are six commonly used filters for fixing dirty lines:

-   [`rekt`](https://gitlab.com/Ututu/rekt)'s `rektlvls`\
    This is basically `FixBrightnessProtect3` and `FixBrightness` in one
    with the additional fact that not the entire frame is processed. Its
    values are quite straightforward. Raise the adjustment values to
    brighten, lower to darken. Set `prot_val` to `None` and it will
    function like `FixBrightness`, meaning the adjustment values will
    need to be changed.
    ```py
    from rekt import rektlvls
    fix = rektlvls(src, rownum=None, rowval=None, colnum=None, colval=None, prot_val=[16, 235])
    ```

    If you'd like to process multiple rows at a time, you can enter a
    list (e.g. `rownum=[0, 1, 2]`).\
    In `FixBrightness` mode, this will perform an adjustment with
    [`std.Levels`](www.vapoursynth.com/doc/functions/levels.html) on the desired row. This means that, in 8-bit,
    every possible value $v$ is mapped to a new value according to the
    following function: $$\begin{aligned}
    &\forall v \leq 255, v\in\mathbb{N}: \\
    &\max\bigg[\min\bigg(\frac{\max(\min(v, \texttt{max\_in}) - \texttt{min\_in}, 0)}{(\texttt{max\_in} - \texttt{min\_in})}\times (\texttt{max\_out} - \texttt{min\_out}) + \texttt{min\_out}, \\
    & 255), 0] + 0.5\end{aligned}$$ For positive `adj_val`,
    $\texttt{max\_in}=235 - \texttt{adj\_val}$. For negative ones,
    $\texttt{max\_out}=235 + \texttt{adj\_val}$. The rest of the values
    stay at 16 or 235 depending on whether they are maximums or
    minimums.\
    `FixBrightnessProtect3` mode takes this a bit further, performing
    (almost) the same adjustment for values between the first
    $\texttt{prot\_val} + 10$ and the second $\texttt{prot\_val} - 10$,
    where it scales linearly. Its adjustment value does not work the
    same, however, so you have to play around with it.

    To illustrate this, let's look at the dirty lines in the black and
    white Blu-ray of Parasite (2019)'s bottom rows:

    ![Parasite b&w source, zoomed via point
    resizing.](Pictures/rektlvls_src.png){width=".9\\textwidth"}

    In this example, the bottom four rows have alternating brightness
    offsets from the next two rows. So, we can use `rektlvls` to raise
    luma in the first and third row from the bottom, and again to lower
    it in the second and fourth:
    ```py
    fix = rektlvls(src, rownum=[803, 802, 801, 800], rowval=[27, -10, 3, -3])
    ```

    In this case, we are in `FixBrightnessProtect3` mode. We aren't
    taking advantage of `prot_val` here, but people usually use this
    mode regardless, as there's always a chance it might help. The
    result:

    ![Parasite b&w source with `rektlvls` applied, zoomed via point
    resizing.](Pictures/rektlvls_fix.png){width=".9\\textwidth"}

-   `awsmfunc`'s `bbmod`\
    This is a mod of the original BalanceBorders function. While it
    doesn't preserve original data nearly as well as `rektlvls`, ti will
    lead to decent results with high `blur` and `thresh` values and is
    easy to use for multiple rows, especially ones with varying
    brightness, where `rektlvls` is no longer useful. If it doesn't
    produce decent results, these can be changed, but the function will
    get more destructive the lower you set the `blur` value. It's also
    significantly faster than the versions in `havsfunc` and `sgvsfunc`
    as only necessary pixels are processed.\

        import awsmfunc as awf
        bb = awf.bbmod(src=clip, left=0, right=0, top=0, bottom=0, thresh=[128, 128, 128], blur=[20, 20, 20], scale_thresh=False, cpass2=False)

    The arrays for `thresh` and `blur` are again y, u, and v values.
    It's recommended to try `blur=999` first, then lowering that and
    `thresh` until you get decent values.\
    `thresh` specifies how far the result can vary from the input. This
    means that the lower this is, the better. `blur` is the strength of
    the filter, with lower values being stronger, and larger values
    being less aggressive. If you set `blur=1`, you're basically copying
    rows. If you're having trouble with chroma, you can try activating
    `cpass2`, but note that this requires a very low `thresh` to be set,
    as this changes the chroma processing significantly, making it quite
    aggressive\
    `bbmod` works by blurring the desired rows, input rows, and
    reference rows within the image using a blurred bicubic kernel,
    whereby the blur amount determines the resolution scaled to accord
    to $\mathtt{\frac{width}{blur}}$. The output is compared using
    expressions and finally merged according to the threshold specified.

    For our example, let's again use Parasite (2019), but the SDR UHD
    this time. It has irregular dirty lines on the top three rows:

    ![Parasite SDR UHD source, zoomed via point
    resizing.](Pictures/bbmod_src.png){width=".9\\textwidth"}

    To fix this, we can apply `bbmod` with a low blur and a low thresh,
    meaning we won't change pixels' values by much:

    ```py
    fix = awf.bbmod(src, top=3, thresh=20, blur=20)
    ```

    ![Parasite SDR UHD source with `bbmod` applied, zoomed via point
    resizing.](Pictures/bbmod_fix.png){width=".9\\textwidth"}

    Our output is already a lot closer to what we assume the source
    should look like. Unlike `rektlvls`, this function is quite quick to
    use, so lazy people (a.. everyone) can use this to fix dirty lines
    before resizing, as the difference won't be noticeable after
    resizing.

-   [`fb`'s](https://github.com/Moiman/vapoursynth-fillborders) `FillBorders`\
    This function pretty much just copies the next column/row in line.
    While this sounds, silly, it can be quite useful when downscaling
    leads to more rows being at the bottom than at the top, and one
    having to fill one up due to YUV420's mod2 height.

    ```py
    fill = core.fb.FillBorders(src=clip, left=0, right=0, bottom=0, top=0, mode="fillmargins")
    ```

    A very interesting use for this function is one similar to applying
    `ContinuityFixer` only to chroma planes, which can be used on gray
    borders or borders that don't match their surroundings no matter
    what luma fix is applied. This can be done with the following
    script:

    ```py
    fill = core.fb.FillBorders(src=clip, left=0, right=0, bottom=0, top=0, mode="fillmargins")
    merge = core.std.Merge(clipa=clip, clipb=fill, weight=[0,1])
    ```

    You can also split the planes and process the chroma planes
    individually, although this is only slightly faster. A wrapper that
    allows you to specify per-plane values for `fb` is `FillBorders` in
    `awsmfunc`.\
    `FillBorders` in `fillmargins` mode works by averaging the previous
    row's pixels; for each pixel, it takes $3\times$ the left pixel of
    the previous row, $2\times$ the middle pixel of the previous row,
    and $3\times$ the right pixel of the previous row, then averaging
    the output.

    To illustrate what a source requiring `FillBorders` might look like,
    let's look at Parasite (2019)'s SDR UHD once again, which requires
    an uneven crop of 277. However, we can't crop this due to chroma
    subsampling, so we need to fill one row. To illustrate this, we'll
    only be looking at the top rows. Cropping with respect to chroma
    subsampling nets us:

    ```py
    crp = src.std.Crop(top=276)
    ```

    ![Parasite source cropped while respecting chroma subsampling,
    zoomed via point
    resizing.](Pictures/fb_src.png){width=".9\\textwidth"}

    Obviously, we want to get rid of the black line at the top, so let's
    use `FillBorders` on it:

    ```py
    fil = crp.fb.FillBorders(top=1, mode="fillmargins")
    ```

    ![Parasite source cropped while respecting chroma subsampling and
    luma fixed via `FillBorders`, zoomed via point
    resizing.](Pictures/fb_luma.png){width=".9\\textwidth"}

    This already looks better, but the orange tones look washed out.
    This is because `FillBorders` only fills one chroma if **two** luma
    are fixed. So, we need to fill chroma as well. To make this easier
    to write, let's use the `awsmfunc` wrapper:

    ```py
    fil = awf.fb(crp, top=1)
    ```

    ![Parasite source cropped while respecting chroma subsampling and
    luma and chroma fixed via `FillBorders`, zoomed via point
    resizing.](Pictures/fb_lumachroma.png){width=".9\\textwidth"}

    Our source is now fixed. Some people may want to resize the chroma
    to maintain original aspect ratio while shifting chroma, but whether
    this is the way to go is not generally agreed upon (personally, I,
    Aicha, disagree with doing this). If you want to go this route:

    ```py
    top = 1
    bot = 1
    new_height = crp.height - (top + bot)
    fil = awf.fb(crp, top=top, bottom=bot)
    out = fil.resize.Spline36(crp.width, new_height, src_height=new_height, src_top=top) 
    ```

-   [`cf`'s](https://gitlab.com/Ututu/VS-ContinuityFixer) `ContinuityFixer`\
    `ContinuityFixer` works by comparing the rows/columns specified to
    the amount of rows/columns specified by `range` around it and
    finding new values via least squares regression. Results are similar
    to `bbmod`, but it creates entirely fake data, so it's preferable to
    use `rektlvls` or `bbmod` with a high blur instead. Its settings
    look as follows:

    ```py
    fix = core.cf.ContinuityFixer(src=clip, left=[0, 0, 0], right=[0, 0, 0], top=[0, 0, 0], bottom=[0, 0, 0], radius=1920)
    ```

    This is assuming you're working with 1080p footage, as `radius`'s
    value is set to the longest set possible as defined by the source's
    resolution. I'd recommend a lower value, although not going much
    lower than $3$, as at that point, you may as well be copying pixels
    (see `FillBorders` below for that). What will probably throw off
    most newcomers is the array I've entered as the values for
    rows/columns to be fixed. These denote the values to be applied to
    the three planes. Usually, dirty lines will only occur on the luma
    plane, so you can often leave the other two at a value of 0. Do note
    an array is not necessary, so you can also just enter the amount of
    rows/columns you'd like the fix to be applied to, and all planes
    will be processed.\
    `ContinuityFixer` works by calculating the least squares
    regression[^29] of the pixels within the radius. As such, it creates
    entirely fake data based on the image's likely edges.\
    One thing `ContinuityFixer` is quite good at is getting rid of
    irregularities such as dots. It's also faster than `bbmod`, but it
    should be considered a backup option.

    Let's look at the `bbmod` example again and apply `ContinuityFixer`:

    ```py
    fix = src.cf.ContinuityFixer(top=[3, 0, 0], radius=6)
    ```

    ![Parasite SDR UHD source with `ContinuityFixer` applied, zoomed via
    point
    resizing.](Pictures/continuityfixer.png){width=".9\\textwidth"}

    Let's compare the second, third, and fourth row for each of these:

    ![Comparison of Parasite SDR UHD source, `bbmod`, and
    `ContinuityFixer`](Pictures/cfx_bbm.png){width=".9\\textwidth"}

    The result is ever so slightly in favor of `ContinuityFixer` here.

-   `edgefixer`'s[^30] `ReferenceFixer`\
    This requires the original version of `edgefixer` (`cf` is just an
    old port of it, but it's nicer to use and processing hasn't
    changed). I've never found a good use for it, but in theory, it's
    quite neat. It compares with a reference clip to adjust its edge fix
    as in `ContinuityFixer`.:

    ```py
    fix = core.edgefixer.Reference(src, ref, left=0, right=0, top=0, bottom=0, radius = 1920)
    ```

One thing that shouldn't be ignored is that applying these fixes (other
than `rektlvls`) to too many rows/columns may lead to these looking
blurry on the end result. Because of this, it's recommended to use
`rektlvls` whenever possible or carefully apply light fixes to only the
necessary rows. If this fails, it's better to try `bbmod` before using
`ContinuityFixer`.

It's important to note that you should *always* fix dirty lines before
resizing, as not doing so will introduce even more dirty lines. However,
it is important to note that, if you have a single black line at an edge
that you would use `FillBorders` on, you should remove that using your
resizer.\

For example, to resize a clip with a single filled line at the top to
$1280\times536$ from $1920\times1080$:

```py
top_crop = 138
bot_crop = 138
top_fill = 1
bot_fill = 0
src_height = src.height - (top_crop + bot_crop) - (top_fill + bot_fill)
crop = core.std.Crop(src, top=top_crop, bottom=bot_crop)
fix = core.fb.FillBorders(crop, top=top_fill, bottom=bot_fill, mode="fillmargins")
resize = core.resize.Spline36(1280, 536, src_top=top_fill, src_height=src_height)
```

If you're dealing with diagonal borders, the proper approach here is to
mask the border area and merge the source with a `FillBorders` call. An
example of this (from the D-Z0N3 encode of Your Name (2016)):

![Example of improper borders from Your Name with brightness lowered.
D-Z0N3 is masked, Geek is unmasked. As such, Geek lacks any resemblance
of grain, while D-Z0N3 keeps it in tact whenever possible. It may have
been smarter to use the `mirror` mode in `FillBorders`, but hindsight is
20/20.](Pictures/improper_borders.png){#fig:25 width="100%"}

Code used by D-Z0N3 (in 16-bit):

```py
mask = core.std.ShufflePlanes(src, 0, vs.GRAY).std.Binarize(43500)
cf = core.fb.FillBorders(src, top=6).std.MaskedMerge(src, mask)
```

Another example of why you should be masking this is in the appendix
under figure [\[fig:26\]](#fig:26){reference-type="ref"
reference="fig:26"}.

Dirty lines can be quite difficult to spot. If you don't immediately
spot any upon examining borders on random frames, chances are you'll be
fine. If you know there are frames with small black borders on each
side, you can use something like the following script[^31]:

```py
def black_detect(clip, thresh=None):
    if thresh:
        clip = core.std.ShufflePlanes(clip, 0, vs.GRAY).std.Binarize(
            "{0}".format(thresh)).std.Invert().std.Maximum().std.Inflate( ).std.Maximum().std.Inflate()
    l = core.std.Crop(clip, right=clip.width / 2)
    r = core.std.Crop(clip, left=clip.width / 2)
    clip = core.std.StackHorizontal([r, l])
    t = core.std.Crop(clip, top=clip.height / 2)
    b = core.std.Crop(clip, bottom=clip.height / 2)
    return core.std.StackVertical([t, b])
```

This script will make values under the threshold value (i.. the black
borders) show up as vertical or horizontal white lines in the middle on
a mostly black background. If no threshold is given, it will simply
center the edges of the clip. You can just skim through your video with
this active. An automated script would be `dirtdtct`[^32], which scans
the video for you.

Other kinds of variable dirty lines are a bitch to fix and require
checking scenes manually.

An issue very similar to dirty lines is bad borders. During scenes with
different crops (e.. IMAX or 4:3), the black borders may sometimes not
be entirely black, or be completely messed up. In order to fix this,
simply crop them and add them back. You may also want to fix dirty lines
that may have occurred along the way:

```py
crop = core.std.Crop(src, left=100, right=100)
clean = core.cf.ContinuityFixer(crop, left=2, right=2, top=0, bottom=0, radius=25)
out = core.std.AddBorders(clean, left=100, right=100)
```
