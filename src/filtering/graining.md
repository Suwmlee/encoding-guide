As grain and dither are some of the hardest things to compress, many
sources will feature very little of this or obviously destroyed grain.
To counteract this or simply to aid with compression of areas with no
grain, it's often beneficial to manually add grain. In this case of
destroyed grain, you will usually want to remove the grain first before
re-applying it. This is especially beneficial with anime, as a lack of
grain can often make it harder for the encoder to maintain gradients.

As we're manually applying grain, we have the option to opt for static
grain. This is almost never noticeable with anime, and compresses a lot
better, hence it's usually the best option for animated content. It is,
however, often quite noticeable in live action content, hence static
grain is not often used in private tracker encodes.

The standard graining function, which the other functions also use, is
`grain.Add`:

    grained = core.grain.Add(clip, var=1, constant=False)

The `var` option here signifies the strength. You probably won't want to
raise this too high. If you find yourself raising it too high, it'll
become noticeable enough to the point where you're better off attempting
to match the grain in order to keep the grain unnoticeable.

The most well-known function for adding grain is `GrainFactory3`. This
function allows you to specify how `grain.Add` should be applied for
three different luma levels (bright, medium, dark). It also scales the
luma with `resize.Bicubic` in order to raise or lower its size, as well
as sharpen it via functions `b` and `c` parameters, which are modified
via the `sharpen` option. It can be quite hard to match here, as you
have to modify size, sharpness, and threshold parameters. However, it
can produce fantastic results, especially for live action content with
more natural grain.

A more automated option is `adaptive_grain`[^39]. This works similarly
to `GrainFactory3`, but instead applies variable amounts of grain to
parts of the frame depending on the overall frame's luma value and
specific areas' luma values. As you have less options, it's easier to
use, and it works fine for anime. The dependency on the overall frame's
average brightness also makes it produce very nice results.

In addition to these two functions, a combination of the two called
`adptvgrnMod`[^40] is available. This adds the sharpness and size
specification options from `GrainFactory3` to\
`adaptive_grain`. As grain is only added to one (usually smaller than
the frame) image in one size, this often ends up being the fastest
function. If the grain size doesn't change for different luma levels, as
is often the case with digitally produced grain, this can lead to better
results than both of the aforementioned functions.

For those curious what this may look like, please refer to the debanding
example from Mirai in figure [3](#fig:3){reference-type="ref"
reference="fig:3"}, as `adptvgrnMod` was used for graining in that
example.
