在处理视频的过程中，通常会遇到一些视觉上的问题，如带状物、变暗的边框等。
由于这些东西在视觉上并不讨人喜欢，而且大多数情况下并不是原创作者的本意，所以最好使用视频过滤器来修复它们。

## 场景过滤

由于每个过滤器都有一定的破坏性，所以最好只在必要时应用它们。这通常是通过`ReplaceFramesSimple`完成的，它在[`RemapFrames`](https://github.com/Irrational-Encoding-Wizardry/Vapoursynth-RemapFrames)插件中。
`RemapFramesSimple`也可以用`Rfs`调用。
另外，可以使用Python解决方案，例如std.Trim和addition。
然而，`RemapFrames`往往更快，特别是对于较大的替换映射集。

让我们看一个对100到200帧和500到750帧应用[`f3kdb`](./debanding.md##neo_f3kdb)解带过滤的例子:

```py
src = core.ffms2.Source("video.mkv")
deband = source.neo_f3kdb.Deband(src)

replaced = core.remap.Rfs(src, deband, mappings="[100 200] [500 750]")
```

在插件和Python方法里有各种封装好的库，特别是前者的[`awsmfunc.rfs`](https://github.com/OpusGang/awsmfunc)和后者的[`lvsfunc.util.replace_frames`](https://lvsfunc.encode.moe/en/latest/#lvsfunc.util.replace_ranges) 。

## 过滤顺序

为了让滤镜正常工作，不至于产生反效果，按正确的顺序应用它们是很重要的。
这一点对于像debanders和graners这样的滤镜尤其重要，因为把它们放在调整大小之前会完全否定它们的效果。

一般可接受的顺序是:
1. 载入视频 Load the source
2. 裁剪 Crop
3. 提高位深 Raise bit depth
4. 除着色 Detint
5. 修复脏线 Fix dirty lines
6. 解块 Deblock
7. 调整大小 Resize
8. 降噪 Denoise
9. 抗锯齿 Anti-aliasing
10. 去晕 Dering (dehalo)
11. 解带 Deband
12. 颗粒 Grain
13. 抖动到输出位深度 Dither to output bit depth

请记住，这只是一个一般的建议。
在某些情况下，你可能想偏离这个建议，例如，如果你使用的是KNLMeansCL这样的快速去噪器，你可以在调整大小之前做这个。
