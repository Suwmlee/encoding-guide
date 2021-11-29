## 0.88伽马的错误

如果你有两个信号源，其中一个明显比另一个亮，那么你的亮的信号源有可能是受到了所谓的伽马缺陷的影响。
如果是这种情况，请执行以下操作（针对16位），看看是否能解决这个问题:

```py
out = core.std.Levels(src, gamma=0.88, min_in=4096, max_in=60160, min_out=4096, max_out=60160, planes=0)
```

<p align="center">
<img src='Pictures/gamma_before.png' onmouseover="this.src='Pictures/gamma_after.png';" onmouseout="this.src='Pictures/gamma_before.png';" />
</p>

不要在低位深度下执行这一操作。较低的比特深度可能并将导致色带。
<p align="center">
<img src='Pictures/gamma_lbd.png' onmouseover="this.src='Pictures/gamma_hbd.png';" onmouseout="this.src='Pictures/gamma_lbd.png';" />
</p>

<details>
<summary>深入解释</summary>
这个错误似乎源于苹果软件。 <a href="https://vitrolite.wordpress.com/2010/12/31/quicktime_gamma_bug/">这篇博文</a>是人们在网上可以找到的关于这个错误的少数提及之一。

其原因可能是软件不必要地试图在NTSC伽马（2.2）和PC伽马（2.5）之间进行转换，因为 \\(\frac{2.2}{2.5}=0.88\\).

为了解决这个问题，每个值都必须提高到0.88的幂，尽管必须进行电视范围标准化:

\\[
v_\mathrm{new} = \left( \frac{v - min_\mathrm{in}}{max_\mathrm{in} - min_\mathrm{in}} \right) ^ {0.88} \times (max_\mathrm{out} - min_\mathrm{out}) + min_\mathrm{out}
\\]

对于那些好奇伽马虫源和源会有什么不同的人来说：除了16、232、233、234和235以外的所有数值都是不同的，最大和最常见的差异是10，从63持续到125。
由于可以打出同等数量的数值，而且该操作通常是在高比特深度下进行的，因此显著的细节损失是不太可能的。
然而，请注意，无论比特深度如何，这都是一个有损失的过程。

</details>

你也可以使用`awsmfunc`中的`fixlvls`封装库，在32位精度下轻松完成这个任务。

## 双重范围压缩

一个类似的问题是双范围压缩。 当这种情况发生时，luma值会在30和218之间。 这可以通过以下方法轻松解决:

```py
out = src.resize.Point(range_in=0, range=1, dither_type="error_diffusion")
out = out.std.SetFrameProp(prop="_ColorRange", intval=1)
```

<p align="center">
<img src='Pictures/double_range_compression0.png' onmouseover="this.src='Pictures/double_range_compression1.png';" onmouseout="this.src='Pictures/double_range_compression0.png';" />
</p>

<details>
<summary>深入解释</summary>
这个问题意味着在编码过程中，某些东西或某些人假定输入是全范围的，尽管它已经是有限范围的。 由于最终的结果通常是有限的范围，这个感知的问题被 "修复 "了。

实际上，我们也可以在<code>std.Levels</code>中做完全相同的事情。 以下是应用于改变范围的数学方法:

\\[
v_\mathrm{new} = \left( \frac{v - min_\mathrm{in}}{max_\mathrm{in} - min_\mathrm{in}} \right) \times (max_\mathrm{out} - min_\mathrm{out}) + min_\mathrm{out}
\\]

对于范围压缩，使用以下数值:
\\[
min_\mathrm{in} = 0 \qquad max_\mathrm{in} = 255 \qquad min_\mathrm{out} = 16 \qquad max_\mathrm{out} = 235
\\]

由于zlib调整器在内部使用32位精度来执行这个，所以最简单的就是使用这些。 然而，这些将改变文件的<code>_ColorRange</code>属性，因此需要使用<code>std.SetFrameProp</code>:

</details>

## 其他不正确的层级

一个密切相关的问题是其他不正确的层级。要解决这个问题，最好是使用一个具有正确层级的参考源，找到与16和235相当的值，然后从那里进行调整（为了清楚起见，在更高的比特深度中这样做）:

```py
out = src.std.Levels(min_in=x, min_out=16, max_in=y, max_out=235)
```

然而，这通常是不可能的。 相反，我们可以做以下数学运算来计算出正确的调整值:
\\[
v = \frac{v_\mathrm{new} - min_\mathrm{out}}{max_\mathrm{out} - min_\mathrm{out}} \times (max_\mathrm{in} - min_\mathrm{in}) + min_\mathrm{in}
\\]

因此，我们可以从要调整的源中选择任何低值，将其设置为 \\(min_\mathrm{in}\\)，在参考源中选择同一像素的值作为 \\(min_\mathrm{out}\\)。对于高值和最大值，我们也是这样做的。然后, 我们用16和235来计算 (再次，最好是高位深度--16位的4096和60160，32位浮动的0和1等等。) 这个 \\(v_\mathrm{new}\\) ，输出值将是上面VapourSynth代码中我们的 \\(x\\) 和 \\(y\\)。

为了说明这一点，让我们使用《燃烧》（2018）的德国和美国蓝光片。 美国的蓝光有正确的水平，而德国的蓝光有不正确的水平。

<p align="center">
<img src='Pictures/burning_usa0.png' onmouseover="this.src='Pictures/burning_ger0.png';" onmouseout="this.src='Pictures/burning_usa0.png';" />
</p>

这里德国的高值是199，而美国的相同像素是207。 对于低值，我们可以找到29和27。 通过这些，我们得到18.6和225.4。 对更多的像素和不同的帧进行这些操作，然后取其平均值，我们得到19和224。 用这些值来调整luma，使我们更接近参考视频的[^1]。

<p align="center">
<img src='Pictures/burning_ger_fixed0.png' onmouseover="this.src='Pictures/burning_usa_fixed0.png';" onmouseout="this.src='Pictures/burning_ger_fixed0.png';" />
</p>

<details>
<summary>深入解释</summary>
看过前面解释的人应该认识到这个函数，因为它是用于水平调整的函数的逆向函数。 我们只需把它反过来，把我们的期望值设为 \(v_\mathrm{new}\)并进行计算。
</details>

## 不恰当的颜色矩阵

如果你有一个不恰当的颜色矩阵的信号源，你可以用以下方法解决这个问题
用以下方法解决。

```py
out = core.resize.Point(src, matrix_in_s='470bg', matrix_s='709')
```

`'470bg'`也就是被称为601的东西。要知道你是否应该这样做，你需要一些参考来源，最好不是网络来源。从技术上讲，你可以识别坏的颜色，并意识到有必要改变矩阵，但在这种情况下，人们应该非常确定。

<p align="center">
<img src='Pictures/burning_matrix_before.png' onmouseover="this.src='Pictures/burning_matrix_after.png';" onmouseout="this.src='Pictures/burning_matrix_before.png';" />
</p>

<details>
<summary>深入解释</summary>
颜色矩阵定义了YCbCr和RGB之间的转换是如何进行的。 由于RGB自然没有任何子采样，剪辑首先从4:2:0转换为4:4:4，然后从YCbCr转换为RGB，然后再进行还原。 在YCbCr到RGB的转换过程中，我们假设是Rec.601矩阵系数，而在转换回来的过程中，我们指定是Rec.709。

之所以很难知道是否假设了不正确的标准，是因为两者涵盖了CIE 1931的类似范围。 色度图应使这一点很明显（包括Rec.2020作为参考）:

<p align="center">
<img src='Pictures/colorspaces.svg'/>
</p>

</details>

## 四舍五入错误

一个轻微的绿色色调可能表明发生了舍入错误。
为了解决这个问题，我们需要在比源文件更高的比特深度上增加半步。

```py
high_depth = vsutil.depth(src, 16)
half_step = high_depth.std.Expr("x 128 +")
out = vsutil.depth(half_step, 8)
```

<p align="center">
<img src='Pictures/rounding_0.png' onmouseover="this.src='Pictures/rounding_1.png';" onmouseout="this.src='Pictures/rounding_0.png';" />
</p>

另外，可以使用[`lvsfunc.misc.fix_cr_tint`](https://github.com/Irrational-Encoding-Wizardry/lvsfunc)代替。
它的默认值与上述内容相当。

<details>
<summary>深入解释</summary>
当工作室从他们的10位母版变成8位时，他们的软件可能总是向下四舍五入（例如1.9会被四舍五入为1）。
我们解决这个问题的方法只是增加了一个8位的半步，如（0.5乘以2 ^ {16 - 8} = 128\）
</details>

## 除着色

请注意，只有在其他方法都失败的情况下，你才应该采用这种方法。

如果你有一个较好的带色调的源，和一个较差的不带色调的源，你想把它去掉，你可以通过[`timecube`](https://github.com/sekrit-twc/timecube)和[DrDre's Color Matching Tool](https://valeyard.net/2017/03/drdres-color-matching-tool-v1-2.php)[^2]来实现。首先，在该工具中添加两张参考截图，导出LUT，保存它，并通过类似的方式添加它:

```py
clip = core.resize.Point(src, matrix_in_s="709", format=vs.RGBS)
detint = core.timecube.Cube(clip, "LUT.cube")
out = core.resize.Point(detint, matrix=1, format=vs.YUV420P16, dither_type="error_diffusion")
```

<p align="center">
<img src='Pictures/detint_before2.png' onmouseover="this.src='Pictures/detint_after2.png';" onmouseout="this.src='Pictures/detint_before2.png';" />
</p>

[^1]: 为了简单起见，这里没有触及色度平面。 这些需要做的工作远远多于luma平面，因为很难找到非常鲜艳的颜色，尤其是像这样的截图。

[^2]: 令人遗憾的是，这个程序是闭源的。 我不知道有什么替代品。
