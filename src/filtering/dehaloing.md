Haloing is a lot what it sounds like: thick, bright lines around edges.
These are quite common with poorly resized content. You may also find
that bad descaling or descaling of bad sources can produce noticeable
haloing. To fix this, you should use either `havsfunc`'s `DeHalo_alpha`
or its already masked counterpart, `FineDehalo`. If using the former,
you'll *have* to write your own mask, as unmasked dehaloing usually
leads to awful results. For a walkthrough of how to write a simple
dehalo mask, check [encode.moe](encode.moe)'s guide[^37]. Note that
`FineDehalo` was written for SD content and its mask might not work very
well with higher resolutions, so it's worth considering writing your own
mask and using `DeHalo_alpha` instead.

As `FineDehalo` is a wrapper around `DeHalo_alpha`, they share some
parameters:

    FineDehalo(src, rx=2.0, ry=None, thmi=80, thma=128, thlimi=50, thlima=100, darkstr=1.0, brightstr=1.0, showmask=0, contra=0.0, excl=True, edgeproc=0.0) # ry defaults to rx
    DeHalo_alpha(clp, rx=2.0, ry=2.0, darkstr=1.0, brightstr=1.0, lowsens=50, highsens=50, ss=1.5)

The explanations on the AviSynth wiki are good enough:
<http://avisynth.nl/index.php/DeHalo_alpha#Syntax_and_Parameters> and
<http://avisynth.nl/index.php/FineDehalo#Syntax_and_Parameters>.

`DeHalo_alpha` works by downscaling the source according to `rx` and
`ry` with a mitchell bicubic ($b=\nicefrac{1}{3},\ c=\nicefrac{1}{3}$)
kernel, scaling back to source resolution with blurred bicubic, and
checking the difference between a minimum and maximum (check
[3.2.14](#masking){reference-type="ref" reference="masking"} if you
don't know what this means) for both the source and resized clip. The
result is then evaluated to a mask according to the following
expressions, where $y$ is the maximum and minimum call that works on the
source, $x$ is the resized source with maximum and minimum, and
everything is scaled to 8-bit:
$$\texttt{mask} = \frac{y - x}{y + 0.0001} \times \left[255 - \texttt{lowsens} \times \left(\frac{y + 256}{512} + \frac{\texttt{highsens}}{100}\right)\right]$$
This mask is used to merge the source back into the resized source. Now,
the smaller value of each pixel is taken for a lanczos resize to
$(\texttt{height} \times \texttt{ss})\times(\texttt{width} \times \texttt{ss})$
of the source and a maximum of the merged clip resized to the same
resolution with a mitchell kernel. The result of this is evaluated along
with the minimum of the merged clip resized to the aforementioned
resolution with a mitchell kernel to find the minimum of each pixel in
these two clips. This is then resized to the original resolution via a
lanczos resize, and the result is merged into the source via the
following:

    if original < processed
        x - (x - y) * darkstr
    else
        x - (x - y) * brightstr
