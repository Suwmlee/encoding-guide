# 位深 Bit Depths: 简介

当你过滤一个画面时，结果被限制在你的比特深度中的可用值。默认情况下，大多数SDR内容为8位，HDR内容为10位。在8位中，你被限制在0和255之间的值。然而，由于大多数视频内容是在有限的范围内，这个范围成为16至235(亮度)和16至240(色度)。

比方说，你想把数值在60到65之间的每个像素提高到0.88的幂。
四舍五入到小数点后3位。

| Original | Raised |
|:--------:|:------:|
| 60       | 36.709 |
| 61       | 37.247 |
| 62       | 37.784 |
| 63       | 38.319 |
| 64       | 38.854 |
| 65       | 39.388 |

由于我们仅限于 0 到 255 之间的整数值，因此将这些四舍五入为 37、37、38、38、39、39。
因此，虽然过滤器不会导致相同的值，但我们将这些四舍五入为相同的值。
这会导致生成一些不需要的 [色带(banding)](debanding.md) 。
例如，提高到 8 位的 0.88 次方与 32 位的更高位深度：

<p align="center"> 
<img src='Pictures/gamma_lbd.png' onmouseover="this.src='Pictures/gamma_hbd.png';" onmouseout="this.src='Pictures/gamma_lbd.png';" />
</p>

为了缓解这种情况，我们在更高的位深度下工作，然后使用所谓的抖动算法在舍入期间添加一些波动并防止产生色带。
通常的位深度是 16 位和 32 位。
虽然 16 位一开始听起来更糟，但差异并不明显，并且 32 位，浮点数而不是整数格式，并不是每个过滤器都支持。

幸运的是，对于那些不在更高位深度下工作的人来说，许多过滤器在内部强制更高的精度并正确地抖动结果。
但是，多次在位深度之间切换会浪费 CPU 周期，并且在极端情况下还会改变图像。

## 更改位深 Changing bit depths

要在更高的位深度下工作，您可以在脚本filter部分的开头和结尾使用`vsutil`库中的 `depth` 方法。
默认情况下，这将使用高质量的抖动算法，并且只需要几次击键：

```py
from vsutil import depth

src = depth(src, 16)

resize = ...

my_great_filter = ...

out = depth(my_great_filter, 8)
```

当您在更高位深度下工作时，重要的是要记住，某些函数可能需要 8 位的参数输入值，而其他函数则需要输入位深度。
如果您在期望 16 位输入的函数中错误地输入假设为 8 位的 255，您的结果将大不相同，因为 255 是 8 位中较高的值，而在 16 位中，这大致相当于 8位 中的 1。

要转换值，你可以使用`vsutil`库中的 `scale_value`方法, 这将有助于处理边缘情况等：

```py
from vsutil import scale_value

v_8bit = 128

v_16bit = scale_value(128, 8, 16)
```

得到 `v_16bit = 32768`, 16 位的中间点。

这对于 32 位浮点数并不那么简单，因为您需要指定是否根据范围缩放偏移量以及缩放亮度还是色度。
这是因为有限范围的亮度值介于 0 和 1 之间，而色度值介于 -0.5 和 +0.5 之间。
通常，您将处理电视范围，因此设置 `scale_offsets=True`:

```py
from vsutil import scale_value

v_8bit = 128

v_32bit_luma = scale_value(128, 8, 32, scale_offsets=True)
v_32bit_chroma = scale_value(128, 8, 32, scale_offsets=True, chroma=True)
```

得到 `v_32bit_luma = 0.5, v_32bit_chroma = 0`.

# Dither Algorithms

TODO
