如果你的去带片段与没有带子的部分相比颗粒很小，你应该考虑使用一个单独的函数来添加匹配的颗粒，这样场景就更容易融合在一起。如果有很多颗粒，你可能要考虑`adptvgrnMod`、adaptive_grain`或`GrainFactory3`；对于不那么明显的颗粒，或者只是对于通常会有很少颗粒的明亮场景，你也可以使用`grain.Add`。颗粒器的主题将在后面的[粒化部分](filtering/graining.md)中进一步阐述。

下面是Mirai的一个例子:

<p align="center">
<img src='Pictures/banding_graining_before.png' onmouseover="this.src='Pictures/banding_graining_after.png';" onmouseout="this.src='Pictures/banding_graining_before.png';"/>
</p>

