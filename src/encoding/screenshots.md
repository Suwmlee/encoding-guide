# 进行截图 Taking Screenshots

在VapourSynth中拍摄简单的屏幕截图是非常容易的。如果你使用的是预览器，你可以用它来代替，但知道如何直接通过VapourSynth进行截图可能还是很有用的。

我们推荐使用 `awsmfunc.ScreenGen`.
它有两个优势:

1. 保存了帧数编号，可以很容易地再次参考，例如，如果你想重新做你的截图。
2. 为你处理适当的转换和压缩，而有些预览器（如VSEdit）可能不是这样的。

要使用`ScreenGen`，创建一个你想要截图的新文件夹，例如 "Screenshots"，和一个名为 "screen.txt "的文件，其中包含你想要截图的帧数,例如:

```
26765
76960
82945
92742
127245
```

然后，在你的VapourSynth脚本的底部，写上

```
awf.ScreenGen(src, "Screenshots", "a")
```

`a`是放在帧编号后面的东西。这对于保持组织性和对截图进行分类是很有用的，同时也可以防止不必要的截图被覆盖。

现在，在命令行中运行你的脚本（或在预览器中重新加载）。

```sh
python vapoursynth_script.vpy
```

完成!
你的截图现在应该在给定的文件夹中。

# 对比源与压制 Comparing Source vs. Encode

将source与你的压制进行比较，可以让潜在的下载者轻松判断你的压制质量。
当采取这些时，重要的是包括你要比较的帧类型，例如比较两个`I`帧将导致极其有利的结果。
你可以使用`awsmfunc.FrameInfo`来做这个。

```py
src = awf.FrameInfo(src, "Source")
encode = awf.FrameInfo(encode, "Encode")
```

如果你想在你的预览器中比较这些，建议将它们交错排列。

```py
out = core.std.Interleave([src, encode])
```

然而，如果你是用`ScreenGen`进行截图，不这样做更容易，只需运行两个`ScreenGen`调用。

```py
src = awf.FrameInfo(src, "Source")
awf.ScreenGen(src, "Screenshots", "a")
encode = awf.FrameInfo(encode, "Encode")
awf.ScreenGen(encode, "Screenshots", "b")
```

注意，在压制的`ScreenGen`中，`"a"`被替换成了`"b"`。
这将允许你按名字对文件夹进行排序，并且每张源截图后面都有一个压制截图，使上传更容易。

### HDR对比 HDR comparisons

对于比较HDR源和HDR压制，建议使用色调图。
这个过程是破坏性的，但你仍然能够分辨出哪些地方被扭曲了，哪些地方被平滑了等等。

为此推荐的函数是`awsmfunc.DynamicTonemap`:

```py
src = awf.DynamicTonemap(src, src_fmt=False, libplacebo=False)
encode = awf.DynamicTonemap(encode, src_fmt=False, libplacebo=False)
```

注意，我们在这里禁用了`src_fmt`和`libplacebo`。
将前者设置为`True`会输出10位4:2:0，这是次优的，因为屏幕截图通常是以8位RGB（没有色度子采样）呈现。
后者被推荐用于比较，因为使用libplacebo会使你的色调图在亮度上更有可能不同，使比较更加困难。

## 选择帧 Choosing frames

在进行截图时，重要的是不要让你的压制看起来有欺骗性的透明。
要做到这一点，你需要确保你截图的是适当的帧类型，以及内容上不同的帧。

幸运的是，这里没有太多需要记住的东西:

* 你的压制的截图应该*永远*是*B*类型帧。
* 你的源的截图*不应该*是*I*类型帧。
* 你的比较应该包括黑暗场景、明亮场景、特写镜头、远景镜头、静态场景、高度动作场景，以及你在这两者之间的任何内容。

# 比较不同的源 Comparing Different Sources

当比较不同的源时，你应该进行类似于比较源与压制的工作。
然而，你可能会遇到不同的裁剪、分辨率或色调，所有这些都会妨碍比较的进行。

对于不同的裁剪，只需将边框加回去:

```py
src_b = src_b.std.AddBorders(left=X, right=Y, top=Z, bottom=A)
```

如果这样做会导致图像内容的偏移，你应该调整大小为4:4:4，这样你就可以添加不均匀的边框。
例如，如果你想在顶部和底部添加1像素高的黑条:

```py
src_b = src_b.resize.Spline36(format=vs.YUV444P8, dither_type="error_diffusion")
src_b = src_b.std.AddBorders(top=1, bottom=1)
```

对于不同的分辨率，建议使用一个简单的Spline36调整大小:

```py
src_b = src_b.resize.Spline36(src_a.width, src_a.height, dither_type="error_diffusion")
```

如果一个源是HDR，另一个是SDR，你可以使用`awsmfunc.DynamicTonemap`:

```py
src_b = awf.DynamicTonemap(src_b, src_fmt=False)
```

关于不同的色调，请参考[色调篇](../filtering/detinting.md).
