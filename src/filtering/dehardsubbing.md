# this needs to be rewritten completely

While this issue is particularly common with anime, it does also occur
in some live action sources, and many music videos or concerts on played
on TV stations with logos, hence it's worth looking at how to remove
hardsubs or logos. For logos, the `Delogo` plugin is well worth
considering. To use it, you're going to need the `.lgd` file of the
logo. You can simply look for this via your favorite search engine and
something should show up. From there, it should be fairly
straightforward what to do with the plugin.

The most common way of removing hardsubs is to compare two sources, one
with hardsubs and one reference source with no hardsubs. The functions
I'd recommend for this are `hardsubmask` and `hardsubmask_fades` from
`kagefunc`[^42]. The former is only useful for sources with black and
white subtitles, while the latter can be used for logos as well as
moving subtitles. Important parameters for both are the `expand`
options, which imply `std.Maximum` calls. Depending on how good your
sources are and how much gets detected, you may want to lower these
values.

Once you have your mask ready, you'll want to merge in your reference
hardsub-less source with the main source. You may want to combine this
process with some tinting, as not all sources will have the same colors.
It's important to note that doing this will yield far better results
than swapping out a good source for a bad one. If you're lazy, these
masks can usually be applied to the entire clip with no problem, so you
won't have to go through the entire video looking for hardsubbed areas.
