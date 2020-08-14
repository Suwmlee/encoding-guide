<p align="center">
<img src='Pictures/dirt_source.png' onmouseover="this.src='Pictures/dirt_filtered.png';" onmouseout="this.src='Pictures/dirt_source.png';"/>
</p>
<p align="center">
<i>Dirty lines from A Silent Voice (2016)'s intro.  On mouseover: fixed with ContinuityFixer and FillBorders.</i>
</p>

Another very common issue is dirty
lines. These are usually found on the borders of video, where a row or
column of pixels exhibits too low or too high luma compared to its
surroundings. Oftentimes, this is the due to improper downscaling,
for example when downscaling after applying borders. Dirty lines can also
occur because the compressionist doesn't know that, while they're working
in 4:2:2, meaning their height doesn't have to be mod2, consumer
products will be 4:2:0, meaning the height has to be mod2, leading to
extra black rows that you can't get rid of during cropping if the main clip isn't placed properly.\
Another form of dirty lines is exhibited when the chroma planes are
present on black bars. Usually, these should be cropped out. The
opposite can also occur, however, where the planes with legitimate luma
information lack chroma information.\

It's important to remember that sometimes your source will have fake lines, meaning ones without legitimate information.  These will usually just mirror the next row/column.  Do not bother fixing these, just crop them instead.

Similarly, if you cannot figure out a proper fix, it's completely reasonable to simply crop off dirty lines or leave them unfixed.  These are usually just single rows, after all, nobody will notice!

There are six commonly used filters for fixing dirty lines:

-   [`rekt`](https://gitlab.com/Ututu/rekt)'s `rektlvls`\
    This is basically `FixBrightnessProtect3` and `FixBrightness` from AviSynth in one,
    with the addition that not the entire frame is processed. Its
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

    To illustrate this, let's look at the dirty lines in the black and
    white Blu-ray of Parasite (2019)'s bottom rows:

    ![Parasite b&w source, zoomed via point
    resizing.](Pictures/rektlvls_src.png)

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
    <p align="center">
    <img src='Pictures/rektlvls_fix.png' onmouseover="this.src='Pictures/rektlvls_src.png';" onmouseout="this.src='Pictures/rektlvls_fix.png';"/>
    </p>
    <details>
    <summary>In-depth function explanation</summary>
    In <code>FixBrightness</code> mode, this will perform an adjustment with
    <a href="www.vapoursynth.com/doc/functions/levels.html"><code>std.Levels</code></a> on the desired row. This means that, in 8-bit,
    every possible value \(v\) is mapped to a new value according to the
    following function: 
    $$\begin{aligned}
    &\forall v \leq 255, v\in\mathbb{N}: \\
    &\max\left[\min\left(\frac{\max(\min(v, \texttt{max_in}) - \texttt{min_in}, 0)}{(\texttt{max_in} - \texttt{min_in})}\times (\texttt{max_out} - \texttt{min_out}) + \texttt{min_out}, 255\right), 0\right] + 0.5
    \end{aligned}$$
    For positive <code>adj_val</code>,
    \(\texttt{max_in}=235 - \texttt{adj_val}\). For negative ones,
    \(\texttt{max_out}=235 + \texttt{adj_val}\). The rest of the values
    stay at 16 or 235 depending on whether they are maximums or
    minimums.

    <code>FixBrightnessProtect3</code> mode takes this a bit further, performing
    (almost) the same adjustment for values between the first
    \\(\texttt{prot_val} + 10\\) and the second \\(\texttt{prot_val} - 10\\),
    where it scales linearly. Its adjustment value does not work the
    same, as it adjusts by \\(\texttt{adj_val} \times 2.19\\).  In 8-bit:
    
    Line brightening:
    $$\begin{aligned}
    &\texttt{if }v - 16 <= 0 \\\\
    &\qquad 16 / \\\\
    &\qquad \texttt{if } 235 - \texttt{adj_val} \times 2.19 - 16 <= 0 \\\\
    &\qquad \qquad 0.01 \\\\
    &\qquad \texttt{else} \\\\
    &\qquad \qquad 235 - \texttt{adj_val} \times 2.19 - 16 \\\\
    &\qquad \times 219 \\\\
    &\texttt{else} \\\\
    &\qquad (v - 16) / \\\\
    &\qquad \texttt{if }235 - \texttt{adj_val} \times 2.19 - 16 <= 0 \\\\
    &\qquad \qquad 0.01 \\\\
    &\qquad \texttt{else} \\\\
    &\qquad \qquad 235 - \texttt{adj_val} \times 2.19 - 16 \\\\
    &\qquad \times 219 + 16
    \end{aligned}$$

    Line darkening:
    $$\begin{aligned}
    &\texttt{if }v - 16 <= 0 \\\\
    &\qquad\frac{16}{219} \times (235 + \texttt{adj_val} \times 2.19 - 16) \\\\
    &\texttt{else} \\\\
    &\qquad\frac{v - 16}{219} \times (235 + \texttt{adj_val} \times 2.19 - 16) + 16 \\\\
    \end{aligned}$$
    
    All of this, which we give the variable \\(a\\), is then protected by (for simplicity's sake, only doing dual <code>prot_val</code>, noted by \\(p_1\\) and \\(p_2\\)):
    $$\begin{aligned}
    & a \times \min \left[ \max \left( \frac{v - p_1}{10}, 0 \right), 1 \right] \\\\
    & + v \times \min \left[ \max \left( \frac{v - (p_1 - 10)}{10}, 0 \right), 1 \right] \times \min \left[ \max \left( \frac{p_0 - v}{-10}, 0\right), 1 \right] \\\\
    & + v \times \max \left[ \min \left( \frac{p_0 + 10 - v}{10}, 0\right), 1\right] 
    \end{aligned}$$
    </details>

-   `awsmfunc`'s `bbmod`\
    This is a mod of the original BalanceBorders function. While it
    doesn't preserve original data nearly as well as `rektlvls`, it will
    lead to decent results with high `blur` and `thresh` values and is
    easy to use for multiple rows, especially ones with varying
    brightness, where `rektlvls` is no longer useful. If it doesn't
    produce decent results, these can be changed, but the function will
    get more destructive the lower you set them. It's also
    significantly faster than the versions in `havsfunc` and `sgvsfunc`,
    as only necessary pixels are processed.\
    
    ```py
    import awsmfunc as awf
    bb = awf.bbmod(src=clip, left=0, right=0, top=0, bottom=0, thresh=[128, 128, 128], blur=[20, 20, 20], planes=[0, 1, 2], scale_thresh=False, cpass2=False)
    ```

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
    aggressive.\

    For our example, I've created fake dirty lines, which we will fix:

    ![Dirty lines](Pictures/dirtfixes0.png)

    To fix this, we can apply `bbmod` with a low blur and a high thresh,
    meaning pixel values can change significantly:

    ```py
    fix = awf.bbmod(src, top=6, thresh=90, blur=20)
    ```
    <p align="center">
    <img src='Pictures/dirtfixes1.png' onmouseover="this.src='Pictures/dirtfixes0.png';" onmouseout="this.src='Pictures/dirtfixes1.png';"/>
    </p>

    Our output is already a lot closer to what we assume the source
    should look like. Unlike `rektlvls`, this function is quite quick to
    use, so lazy people (i.e. everyone) can use this to fix dirty lines
    before resizing, as the difference won't be noticeable after
    resizing.
    
    While you can use `rektlvls` on as many rows/columns as necessary, the same doesn't hold true for `bbmod`.  Unless you are resizing after, you should only use `bbmod` on two rows/pixels for low `blur` values (\\(\approx 20\\)) or three for higher `blur` values.  If you are resizing after, you can change the maximum value according to:
    \\[
    max_\mathrm{resize} = max \times \frac{resolution_\mathrm{source}}{resolution_\mathrm{resized}}
    \\]
    <details>
    <summary>In-depth function explanation</summary>
    <code>bbmod</code> works by blurring the desired rows, input rows, and
    reference rows within the image using a blurred bicubic kernel,
    whereby the blur amount determines the resolution scaled to accord
    to \(\mathtt{\frac{width}{blur}}\). The output is compared using
    expressions and finally merged according to the threshold specified.

    The function re-runs one function for the top border for each side by flipping and transposing.  As such, this explanation will only cover fixing the top.
    
    First, we double the resolution without any blurring (\\(w\\) and \\(h\\) are input clip's width and height):
    \\[
    clip_2 = \texttt{resize.Point}(clip, w\times 2, h\times 2)
    \\]
    <p align="center">
    <img src='Pictures/bbmod0_0.png' />
    </p>
    
    Now, the reference is created by cropping off double the to-be-fixed number of rows.  We set the height to 2 and then match the size to the double res clip:
    \\[\begin{align}
    clip &= \texttt{CropAbs}(clip_2, \texttt{width}=w \times 2, \texttt{height}=2, \texttt{left}=0, \texttt{top}=top \times 2) \\\\
    clip &= \texttt{resize.Point}(clip, w \times 2, h \times 2)
    \end{align}\\]
    <p align="center">
    <img src='Pictures/bbmod0_1.png' />
    </p>
    
    Before the next step, we determine the \\(blurwidth\\):
    \\[
    blurwidth = \max \left( 8, \texttt{floor}\left(\frac{w}{blur}\right)\right)
    \\]
    In our example, we get 8.
    
    Now, we use a blurred bicubic resize to go down to \\(blurwidth \times 2\\) and back up:
    \\[\begin{align}
    referenceBlur &= \texttt{resize.Bicubic}(clip, blurwidth \times 2, top \times 2, \texttt{b}=1, \texttt{c}=0) \\\\
    referenceBlur &= \texttt{resize.Bicubic}(referenceBlur, w \times 2, top \times 2, \texttt{b}=1, \texttt{c}=0)
    \end{align}\\]
    <p align="center">
    <img src='Pictures/bbmod0_2.png' />
    </p>
    <p align="center">
    <img src='Pictures/bbmod0_3.png' />
    </p>
    
    Then, crop the doubled input to have height of \\(top \times 2\\):
    \\[
    original = \texttt{CropAbs}(clip_2, \texttt{width}=w \times 2, \texttt{height}=top \times 2)
    \\]
    <p align="center">
    <img src='Pictures/bbmod0_4.png' />
    </p>
    
    Prepare the original clip using the same bicubic resize downwards:
    \\[
    clip = \texttt{resize.Bicubic}(original, blurwidth \times 2, top \times 2, \texttt{b}=1, \texttt{c}=0)
    \\]
    <p align="center">
    <img src='Pictures/bbmod0_5.png' />
    </p>
    
    Our prepared original clip is now also scaled back down:
    \\[
    originalBlur = \texttt{resize.Bicubic}(clip, w \times 2, top \times 2, \texttt{b}=1, \texttt{c}=0)
    \\]
    <p align="center">
    <img src='Pictures/bbmod0_6.png' />
    </p>
    
    Now that all our clips have been downscaled and scaled back up, which is the blurring process that approximates what the actual value of the rows should be, we can compare them and choose how much of what we want to use.  First, we perform the following expression (\\(x\\) is \\(original\\), \\(y\\) is \\(originalBlur\\), and \\(z\\) is \\(referenceBlur\\)):
    \\[
    \max \left[ \min \left( \frac{z - 16}{y - 16}, 8 \right), 0.4 \right] \times (x + 16) + 16
    \\]
    The input here is:
    \\[
    balancedLuma = \texttt{Expr}(\texttt{clips}=[original, originalBlur, referenceBlur], \texttt{"z 16 - y 16 - / 8 min 0.4 max x 16 - * 16 +"})
    \\]
    <p align="center">
    <img src='Pictures/bbmod0_7.png' />
    </p>
    
    What did we do here?  In cases where the original blur is low and supersampled reference's blur is high, we did:
    \\[
    8 \times (original + 16) + 16
    \\]
    This brightens the clip significantly.  Else, if the original clip's blur is high and supersampled reference is low, we darken:
    \\[
    0.4 \times (original + 16) + 16
    \\]
    In normal cases, we combine all our clips:
    \\[
    (original + 16) \times \frac{originalBlur - 16}{referenceBlur - 16} + 16
    \\]
    
    We add 128 so we can merge according to the difference between this and our input clip:
    \\[
    difference = \texttt{MakeDiff}(balancedLuma, original)
    \\]
    
    Now, we compare to make sure the difference doesn't exceed \\(thresh\\):
    \\[\begin{align}
    difference &= \texttt{Expr}(difference, "x thresh > thresh x ?") \\\\
    difference &= \texttt{Expr}(difference, "x thresh < thresh x ?")
    \end{align}\\]
    
    These expressions do the following:
    \\[\begin{align}
    &\texttt{if }difference >/< thresh:\\\\
    &\qquad    thresh\\\\
    &\texttt{else}:\\\\
    &\qquad    difference
    \end{align}\\]

    This is then resized back to the input size and merged using <code>MergeDiff</code> back into the original and the rows are stacked onto the input.  The output resized to the same res as the other images:

    <p align="center">
    <img src='Pictures/bbmod0_9.png' />
    </p>
    </details>

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

    Note that you should only ever fill single columns/rows with `FillBorders`.  If you have more black lines, crop them!  If there are frames requiring different crops in the video, don't fill these up.  More on this at the end of this chapter.

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
    resizing.](Pictures/fb_src.png)

    Obviously, we want to get rid of the black line at the top, so let's
    use `FillBorders` on it:

    ```py
    fil = crp.fb.FillBorders(top=1, mode="fillmargins")
    ```
    <p align="center">
    <img src='Pictures/fb_luma.png' onmouseover="this.src='Pictures/fb_src.png';" onmouseout="this.src='Pictures/fb_luma.png';"/>
    </p>

    This already looks better, but the orange tones look washed out.
    This is because `FillBorders` only fills one chroma if **two** luma
    are fixed. So, we need to fill chroma as well. To make this easier
    to write, let's use the `awsmfunc` wrapper:

    ```py
    fil = awf.fb(crp, top=1)
    ```
    <p align="center">
    <img src='Pictures/fb_lumachroma.png' onmouseover="this.src='Pictures/fb_luma.png';" onmouseout="this.src='Pictures/fb_lumachroma.png';"/>
    </p>

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
    <details>
    <summary>In-depth function explanation</summary>
    <code>FillBorders</code> has three modes, although we only really care about mirror and fillmargins.
    The mirror mode literally just mirrors the previous pixels.  Contrary to the third mode, repeat, it doesn't just mirror the final row, but the rows after that for fills greater than 1.  This means that, if you only fill one row, these modes are equivalent.  Afterwards, the difference becomes obvious.
    
    In fillmargins mode, it works a bit like a convolution, whereby it does a [2, 3, 2] of the next row's pixels, meaning it takes 2 of the left pixel, 3 of the middle, and 2 of the right, then averages.  For borders, it works slightly differently: the leftmost pixel is just a mirror of the next pixel, while the eight rightmost pixels are also mirrors of the next pixel.  Nothing else happens here.
    </details>

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
    lower than 3, as at that point, you may as well be copying pixels
    (see `FillBorders` below for that). What will probably throw off
    most newcomers is the array I've entered as the values for
    rows/columns to be fixed. These denote the values to be applied to
    the three planes. Usually, dirty lines will only occur on the luma
    plane, so you can often leave the other two at a value of 0. Do note
    an array is not necessary, so you can also just enter the amount of
    rows/columns you'd like the fix to be applied to, and all planes
    will be processed.\
    
    As `ContinuityFixer` is less likely to keep original data in tact, it's recommended to prioritize `bbmod` over it.

    Let's look at the `bbmod` example again and apply `ContinuityFixer`:

    ```py
    fix = src.cf.ContinuityFixer(top=[6, 6, 6], radius=10)
    ```
    <p align="center">
    <img src='Pictures/dirtfixes2.png' onmouseover="this.src='Pictures/dirtfixes0.png';" onmouseout="this.src='Pictures/dirtfixes2.png';"/>
    </p>

    Let's compare this with the bbmod fix (remember to mouse-over to compare):

    <p align="center">
    <img src='Pictures/dirtfixes2.png' onmouseover="this.src='Pictures/dirtfixes1.png';" onmouseout="this.src='Pictures/dirtfixes2.png';"/>
    </p>
    The result is ever so slightly in favor of <code>ContinuityFixer</code> here.
 
    Just like `bbmod`, `ContinuityFixer` shouldn't be used on more than two rows/columns.  Again, if you're resizing, you can change this maximum accordingly:
    \\[
    max_\mathrm{resize} = max \times \frac{resolution_\mathrm{source}}{resolution_\mathrm{resized}}
    \\]   
    <details>
    <summary>In-depth function explanation</summary>
    <code>ContinuityFixer</code> works by calculating the <a href=https://en.wikipedia.org/wiki/Least_squares>least squares
    regression</a> of the pixels within the radius. As such, it creates
    entirely fake data based on the image's likely edges.  No special explanation here.
    </details>

-   [`edgefixer`'s](https://github.com/sekrit-twc/EdgeFixer) `ReferenceFixer`\
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
\\(1280\times536\\) from \\(1920\times1080\\):

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
example of this (from the Your Name (2016)):

<p align="center">
<img src='Pictures/improper_borders0.png' onmouseover="this.src='Pictures/improper_borders1.png';" onmouseout="this.src='Pictures/improper_borders0.png';"/>
</p>

Fix compared with unmasked and contrast adjusted for clarity:
<p align="center">
<img src='Pictures/improper_borders_adjusted1.png' onmouseover="this.src='Pictures/improper_borders_adjusted2.png';" onmouseout="this.src='Pictures/improper_borders_adjusted1.png';"/>
</p>

Code used (note that this was detinted after):
```py
mask = core.std.ShufflePlanes(src, 0, vs.GRAY).std.Binarize(43500)
cf = core.fb.FillBorders(src, top=6, mode="mirror").std.MaskedMerge(src, mask)
```

Dirty lines can be quite difficult to spot. If you don't immediately
spot any upon examining borders on random frames, chances are you'll be
fine. If you know there are frames with small black borders on each
side, you can use something like the [following script](https://gitlab.com/snippets/1834089):

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

This script will make values under the threshold value (i.e. the black
borders) show up as vertical or horizontal white lines in the middle on
a mostly black background. If no threshold is given, it will simply
center the edges of the clip. You can just skim through your video with
this active. An automated alternative would be [`dirtdtct`](https://git.concertos.live/AHD/awsmfunc/src/branch/master/awsmfunc/detect.py), which scans
the video for you.

Other kinds of variable dirty lines are a bitch to fix and require
checking scenes manually.

An issue very similar to dirty lines is unwanted borders. During scenes with
different crops (e.g. IMAX or 4:3), the black borders may sometimes not
be entirely black, or be completely messed up. In order to fix this,
simply crop them and add them back. You may also want to fix dirty lines
that may have occurred along the way:

```py
crop = core.std.Crop(src, left=100, right=100)
clean = core.cf.ContinuityFixer(crop, left=2, right=2, top=0, bottom=0, radius=25)
out = core.std.AddBorders(clean, left=100, right=100)
```
