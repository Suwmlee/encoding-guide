首先，您需要选择视频文件的一个较小区域作为参考，因为对整个内容进行测试将花费很长时间。
推荐的方法是使用 `awsmfunc`里的`SelectRangeEvery`:

    import awsmfunc as awf
    out = awf.SelectRangeEvery(clip, every=15000, length=250, offset=[1000, 5000])

这里，第一个数字是节之间的偏移量，第二个数字是每个节的长度，偏移数组是从开始到结束的偏移量。

您需要使用相当长的剪辑（通常为几千帧），其中包括黑暗、明亮、静态和动作场景，
但是，它们的分布应该与它们在整个视频中的分布大致相同。

在测试设置时，您应该始终使用 2-pass 编码，因为许多设置会显着改变 CRF 为您提供的比特率。
对于最终编码，两者都很好，尽管 CRF 更快。

要找出最佳设置，请将它们相互对比并与源进行比较。 你可以单独这样做，或者在`awsmfunc'文件夹中交错排列一个文件夹。你通常也想给它们贴上标签，这样你就可以让你真正知道你在看哪个片段。

    # Load the files before this
    src = awf.FrameInfo(src, "Source")
    test1 = awf.FrameInfo(test1, "Test 1")
    test2 = awf.FrameInfo(test2, "Test 2")
    out = core.std.Interleave([src, test1, test2])

    # You can also place them all in the same folder and do
    src = awf.FrameInfo(src, "Source")
    folder = "/path/to/settings_folder"
    out = awf.InterleaveDir(src, folder, PrintInfo=True, first=extract, repeat=True)

如果你使用`yuuno`，你可以使用下面的iPython魔法来获得 悬停在预览屏幕上，使预览在两个源之间切换
屏幕。

    %vspreview --diff
    clip_A = core.ffms2.Source("settings/crf/17.0")
    clip_A.set_output()
    clip_B = core.ffms2.Source("settings/crf/17.5")
    clip_B.set_output(1)

通常情况下，你会想先测试一下比特率。只要在几个不同的CRF下进行编码，并与源文件进行比较，找到与源文件无法区分的最高CRF值。
现在，对数值进行四舍五入，最好是向下，并切换到2-pass。对于标准测试，测试qcomp（间隔为0.05），大内部的aq强度的aq模式（例如，对于一个aq模式做测试，aq强度从0.6到1.0，间隔为0.2），aq强度（间隔为0.05），merange（32，48和64），psy-rd（间隔为0.05），ipratio/bratio（间隔为0.05，距离保持为0.10），然后deblock（间隔为1）。
如果你认为 mbtree 有所帮助（即你正在对动画进行编码），请在打开 mbtree 的情况下重做这个过程。你可能不会想怎么改变顺序，但肯定可以这样做。

对于x265，顺序应该是qcomp、aq-mode、aq-strength、psy-rd、psy-rdoq、ipratio和pbratio，然后是deblock。

如果你想要一点额外的效率，你可以在你最终决定的每个设置的数值周围用较小的间隔再次进行测试。建议在你已经对每个设置做了一次测试之后再做，因为它们确实对彼此都有轻微的影响。

一旦你完成了用2-pass的测试设置，就切换回CRF，并重复寻找最高透明CRF值的过程。