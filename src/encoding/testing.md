To start off, you'll want to select a smaller region of your video file
to use as reference, since testing on the entire thing would take
forever. The recommended way of doing this is by using `awsmfunc`'s
`SelectRangeEvery`:

    import awsmfunc as awf
    out = awf.SelectRangeEvery(clip, every=15000, length=250, offset=[1000, 5000])

Here, the first number is the offset between sections, the second one is
the length of each section, and the offset array is the offset from
start and end.

You'll want to use a decently long clip (a couple thousand frames
usually) that includes both dark, bright, static, and action scenes,
however, these should be roughly as equally distributed as they are in
the entire video.

When testing settings, you should always use 2-pass encoding, as many
settings will substantially change the bitrate CRF gets you. For the
final encode, both are fine, although CRF is faster.

To find out what setting is best, compare them all to each other and the
source. You can do so by interleaving them either individually or by a
folder via `awsmfunc`. You'll usually also want to label them, so you
actually know which clip you're looking at:

    # Load the files before this
    src = awf.FrameInfo(src, "Source")
    test1 = awf.FrameInfo(test1, "Test 1")
    test2 = awf.FrameInfo(test2, "Test 2")
    out = core.std.Interleave([src, test1, test2])

    # You can also place them all in the same folder and do
    src = awf.FrameInfo(src, "Source")
    folder = "/path/to/settings_folder"
    out = awf.InterleaveDir(src, folder, PrintInfo=True, first=extract, repeat=True)

If you're using `yuuno`, you can use the following iPython magic to get
the preview to switch between two source by hovering over the preview
screen:

    %vspreview --diff
    clip_A = core.ffms2.Source("settings/crf/17.0")
    clip_A.set_output()
    clip_B = core.ffms2.Source("settings/crf/17.5")
    clip_B.set_output(1)

Usually, you'll want to test for the bitrate first. Just encode at a
couple different CRFs and compare them to the source to find the highest
CRF value that is indistinguishable from the source. Now, round the
value, preferably down, and switch to 2-pass. For standard testing, test
qcomp (intervals of 0.05), aq-modes with aq-strengths in large internals
(e.. for one aq-mode do tests with aq-strengths ranging from 0.6 to 1.0
in intervals of 0.2), aq-strength (intervals of 0.05), merange (32, 48,
and 64), psy-rd (intervals of 0.05), ipratio/pbratio (intervals of 0.05
with distance of 0.10 maintained), and then deblock (intervals of 1). If
you think mbtree could help (i.. you're encoding animation), redo this
process with mbtree turned on. You probably won't want to change the
order much, but it's certainly possible to do so.

For x265, the order should be qcomp, aq-mode, aq-strength, psy-rd,
psy-rdoq, ipratio and pbratio, and then deblock.

If you want that little extra efficiency, you can redo the tests again
with smaller intervals surrounding the areas around which value you
ended up deciding on for each setting. It's recommended to do this after
you've already done one test of each setting, as they do all have a
slight effect on each other.

Once you're done testing the settings with 2-pass, switch back to CRF
and repeat the process of finding the highest transparent CRF value.
