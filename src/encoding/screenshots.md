# 截图 Taking Screenshots

在VapourSynth中拍摄简单的屏幕截图是非常容易的。
如果你使用的是previewer，你可以用它来代替，，但知道如何直接通过VapourSynth进行截图可能还是很有用的。

我们推荐使用 `awsmfunc.ScreenGen`.
它有两个优势:

1. 保存了帧数编号，可以很容易再次引用，例如，如果你想重新做你的截图。
2. 为你处理适当的转换和压缩，而有些previewer（例如VSEdit）可能不是这样的。

要使用`ScreenGen`，首先创建一个你想要截图的新文件夹，例如 "Screenshots"，和一个名为 "screen.txt "的文件，其中包含你想要截图的帧数,例如:

```
26765
76960
82945
92742
127245
```

然后，在你的VapourSynth脚本最后，写上

```
awf.ScreenGen(src, "Screenshots", "a")
```

`a`是放在帧编号后面的东西。
这对于保持文件结构和对截图进行分类是很有用的，同时也可以防止不必要的截图被覆盖。

现在，在命令行中运行你的脚本（或在previewer中重新加载）。

```sh
python vapoursynth_script.vpy
```

完成!
你的截图现在应该在给定的文件夹中。

# 对比源与压制 Comparing Source vs. Encode

将source与你的压制作品进行比较，可以让潜在的下载者轻松判断你的压制质量。
在进行比较时，包含的比较帧的类型非常重要，例如比较两个`I`帧将导致极其有利的结果。
你可以使用`awsmfunc.FrameInfo`来做这个:

```py
src = awf.FrameInfo(src, "Source")
encode = awf.FrameInfo(encode, "Encode")
```

如果你想在你的previewer中比较这些，建议将它们交错排列。

```py
out = core.std.Interleave([src, encode])
```

然而，如果你是用`ScreenGen`进行截图，将会非常简单，只需调用两次`ScreenGen`:

```py
src = awf.FrameInfo(src, "Source")
awf.ScreenGen(src, "Screenshots", "a")
encode = awf.FrameInfo(encode, "Encode")
awf.ScreenGen(encode, "Screenshots", "b")
```

注意，在压制的`ScreenGen`中，`"a"`被替换成了`"b"`。
这将允许你按名字对文件夹进行排序，并且每张源截图后面都有一个压制截图，使上传更容易。

### HDR对比 HDR comparisons

对于比较HDR源与HDR压制，建议进行色调映射。
这个过程具有破坏性，但你仍然能够分辨出哪些地方被扭曲了，哪些地方被平滑了等等。

推荐使用`awsmfunc.DynamicTonemap`方法:

```py
src = awf.DynamicTonemap(src)
encode = awf.DynamicTonemap(encode, reference=src)
```

第二个色调映射方法中的 `reference=src` 确保了两个色调映射输出的一致性。

## 选择帧 Choosing frames

在进行截图时，不要让你的压制看起来有欺骗性的透明是非常重要的。
要做到这一点，你需要确保你截图的是较多适当的帧类型，以及内容上不同的帧。

幸运的是，这里没有太多需要记住的东西:

* 你的压制截图图应该*永远*是*B*类型帧。
* 你的源截图*不应该*是*I*类型帧。
* 你的比较截图应该包括黑暗场景、明亮场景、特写镜头、远景镜头、静态场景、高度动作场景，以及介于两者之间的任何场景

# 比较不同的源 Comparing Different Sources

当比较不同的源时，你应该进行类似于比较源与压制的步骤。
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

对于不同的分辨率，推荐使用Spline36方法调整大小[^1]:

```py
src_b = src_b.resize.Spline36(src_a.width, src_a.height, dither_type="error_diffusion")
```

如果一个源是HDR，另一个是SDR，你可以使用`awsmfunc.DynamicTonemap`方法:

```py
src_b = awf.DynamicTonemap(src_b)
```

关于不同的色调，可以参考[色调篇](../filtering/detinting.md)。

[^1]: 重要的是要确保你在这里调整到适当的分辨率；如果你为1080p编码进行比较，你显然会以1080p进行比较，而如果你是为了弄清楚哪个来源更好而进行比较，你会想把低分辨率的来源提升到与高分辨率相匹配。
