# Black & White Clips: Working in GRAY

由于YUV格式将luma保存在一个单独的平面内，在处理黑白电影时，我们会变得更加轻松，因为我们可以提取luma平面并只在该平面上工作。

```py
y = src.std.ShufflePlanes(0, vs.GRAY)
```

`vsutil`中的`get_y`函数做了同样的事情。
有了我们的`y`片段，我们可以执行没有mod2限制的功能。
例如，我们可以进行奇数裁剪:

```py
crop = y.std.Crop(left=1)
```

此外，由于滤镜只应用在一个平面上，这可以加快我们的脚本速度。
我们也不必担心像`f3kdb`这样的滤镜会以不需要的方式改变我们的色度平面，例如使它们产生颗粒。

然而，当我们完成我们的剪辑后，我们通常想把剪辑导出为YUV。
这里有两个选项:

### 1. 使用假色度

使用假的色度是快速和简单的，其优点是在源中任何意外的色度偏移（例如色度颗粒）将被删除。
所有这一切都需要恒定的色度（意味着没有色调变化）和mod2 luma。

最简单的选项是`u = v = 128`（8位）:

```py
out = y.resize.Point(format=vs.YUV420P8)
```

如果你有一个不均匀的luma，只需用`awsmfunc.fb`来填充它。
假设你想对左边进行填充:

```py
y = core.std.StackHorizontal([y.std.BlankClip(width=1), y])
y = awf.fb(y, left=1)
out = y.resize.Point(format=vs.YUV420P8)
```

另外，如果你的源的色度不是中性灰色，可以使用`std.BlankClip`:

```py
blank = y.std.BlankClip(color=[0, 132, 124])
out = core.std.ShufflePlanes([y, blank], [0, 1, 2], vs.YUV)
```

### 2. 使用原始色度（必要时调整大小）

这样做的好处是，如果有实际重要的色度信息（例如轻微的棕褐色色调），这将被保留下来。
只要在你的素材上使用`ShufflePlanes`就可以了:

```py
out = core.std.ShufflePlanes([y, src], [0, 1, 2], vs.YUV)
```

然而，如果你已经调整了尺寸或裁剪，这就变得有点困难了。
你可能必须适当地移动或调整色度（见[色度重采样章节](.../filtering/chroma_rs.md)的解释）。

如果你已经裁剪了，就提取并相应地移位。我们将使用`vsutil`的`split`和`join`来提取和合并平面:

```py
y, u, v = split(src)
crop_left = 1
y = y.std.Crop(left=crop_left)
u = u.resize.Spline36(src_left=crop_left / 2)
v = v.resize.Spline36(src_left=crop_left / 2)
out = join([y, u, v])
```

如果你已经调整了尺寸，你需要转移和调整色度的大小:

```py
y, u, v = split(src)
w, h = 1280, 720
y = y.resize.Spline36(w, h)
u = u.resize.Spline36(w / 2, h / 2, src_left=.25 - .25 * src.width / w)
v = v.resize.Spline36(w / 2, h / 2, src_left=.25 - .25 * src.width / w)
out = join([y, u, v])
```

结合裁剪和移位，据此我们垫高裁剪并使用`awsmfunc.fb`来创建一个假线:

```py
y, u, v = split(src)
w, h = 1280, 720
crop_left, crop_bottom = 1, 1

y = y.std.Crop(left=crop_left, bottom=crop_bottom)
y = y.resize.Spline36(w - 1, h - 1)
y = core.std.StackHorizontal([y.std.BlankClip(width=1), y])
y = awf.fb(left=1)

u = u.resize.Spline36(w / 2, h / 2, src_left=crop_left / 2 + (.25 - .25 * src.width / w), src_height=u.height - crop_bottom / 2)

v = v.resize.Spline36(w / 2, h / 2, src_left=crop_left / 2 + (.25 - .25 * src.width / w), src_height=u.height - crop_bottom / 2)

out = join([y, u, v])
```

如果你不明白这里到底发生了什么，遇到这样的情况，请向更有经验的人求助。

