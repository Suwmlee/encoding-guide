Although not necessary to work in if you're exporting in the same bit
depth as your source, working in high bit depth and dithering down at
the end of your filter chain is recommended in order to avoid rounding
errors, which can lead to artifacts such as banding (an example is in
figure [20](#fig:flvls){reference-type="ref" reference="fig:flvls"}).
Luckily, if you choose not to write your script in high bit depth, most
plugins will work in high bit depth internally. As dithering is quite
fast and higher depths do lead to better precision, there's usually no
reason not to work in higher bit depths other than some functions
written for 8-bit being slightly slower.

If you'd like to learn more about dithering, the Wikipedia page[^12] is
quite informative. There are also a lot of research publications worth
reading. What you need to understand here is that your dither method
only matters if there's an actual difference between your source and the
filtering you perform. As dither is an alternative of sorts to rounding
to different bit depths, only offsets from actual integers will have
differences. Some algorithms might be better at different things from
others, hence it can be worth it to go with non-standard algorithms. For
example, if you want to deband something and export it in 8-bit but are
having issues with compressing it properly, you might want to consider
ordered dithering, as it's known to perform slightly better in this case
(although it doesn't look as nice). To do this, use the following code:

    source_16 = fvf.Depth(src, 16)

    deband = core.f3kdb.Deband(source_16, output_depth=16)

    out = fvf.Depth(deband, 8, dither='ordered')

Again, this will only affect the actual debanded area. This isn't really
recommended most of the time, as ordered dither is rather unsightly, but
it's certainly worth considering if you're having trouble compressing a
debanded area. You should obviously be masking and adjusting your
debander's parameters, but more on that later.

In order to dither up or down, you can use the `Depth` function within
either `fvsfunc`[^13] (fvf) or `mvsfunc`[^14] (mvf). The difference
between these two is that fvf uses internal resizers, while mvf uses
internal whenever possible, but also supports `fmtconv`, which is slower
but has more dither (and resize) options. Both feature the standard
Filter Lite error\_diffusion dither type, however, so if you just roll
with the defaults, I'd recommend fvf. To illustrate the difference
between good and bad dither, some examples are included in the appendix
under figure [19](#fig:12){reference-type="ref" reference="fig:12"}. Do
note you may have to zoom in quite far to spot the difference. Some PDF
viewers may also incorrectly output the image.

I'd recommend going with Filter Lite (fvf's default or
`mvf.Depth(dither=3)`, also the default) most of the time. Others like
Ostromoukhov (`mvf.Depth(dither=7)`), void and cluster
(`fmtc.bitdepth(dither=8)`), standard Bayer ordered
(`fvf.Depth(dither=’ordered’)` or `mvf.Depth(dither=0)`) can also be
useful sometimes. Filter Lite will usually be fine, though.
