## DeHalo_alpha

<details>
<summary>old function explanation</summary>
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
</details>

### Masking
