# 介绍

本指南的目的是为对制作高质量压制感兴趣的新人提供一个起点，同时也为有经验的压制人员提供参考。
因此，在介绍了大多数功能和解释了它们的用途之后，可以找到一个深入的解释。
这些都是为好奇者提供的简单补充，不是使用相关方法的必要阅读。

# 译者前言

该指南原始档案: [mdbook-guide](https://git.concertos.live/Encode_Guide/mdbook-guide)    
指南主要贡献者Aicha，知名压制组组员，贡献很多高质量压制作品。   
指南内容丰富，结合作者的自身经验深入浅出讲解压制技巧与流程，虽然部分章节仍未完成，但不可否认这仍是一份难得的压制指南。

翻译此指南旨在分享，希望可以帮助他人，了解，学习高质量压制，压制出优秀的作品。部分名词翻译不到位，还请谅解。

# 术语

为此，涉及有关视频的基本术语和知识。
一些解释将被大大简化，因为在 Wikipedia、AviSynth wiki 等网站上应该有很多页面对这些主题进行解释。

## 视频

消费类视频产品通常存储在 YCbCr 中，该术语通常与 YUV 互换使用。在本指南中，我们将主要使用术语 YUV，因为 VapourSynth 格式编写也是 YUV。

YUV 格式的内容将信息分为三个平面: Y, 指代 luma, 表示亮度, U 和 V，分别表示色度平面。
这些色度平面代表颜色之间的偏移，其中平面的中间值是中性点。

这个色度信息通常是二次采样的, 这意味着它存储在比亮度平面更低的帧大小中。
几乎所有消费者视频都采用 4:2:0 格式，这意味着色度平面是亮度平面大小的一半。
[关于色度子采样的Wikipedia](https://en.wikipedia.org/wiki/Chroma_subsampling) 应该足以解释这是如何工作的。
由于我们通常希望保持 4:2:0 格式，因此我们的框架尺寸受到限制，因为亮度平面的大小必须被 2 整除。
这意味着我们不能进行不均匀的裁剪或将大小调整为不均匀的分辨率。
但是，在必要时，我们可以在每个平面中单独处理，这些将会在 [过滤章节](filtering/filtering.md) 进行解释。

此外，我们的信息必须以特定的精度存储。
通常，我们处理每平面 8-bit 的精度。
但是，对于UHD蓝光来说，每平面标准是 10-bit 的精度。
这意味着每个平面的可能值范围从 0 到 \\(2^8 - 1 = 255\\)。
在 [位深章节](filtering/bit_depths.md), 我们将介绍在更高的位深精度下工作，以提高过滤过程中的精度

## VapourSynth

我们将通过 Python 使用 VapourSynth 框架来进行裁剪、移除不需要的黑色边框、调整大小和消除源中不需要的产物。
虽然使用Python听起来令人生畏，但没有经验的人也不必担心，因为我们只做非常基本的事情。

关于 VapourSynth 配置的资料不计其数，例如 [the Irrational Encoding Wizardry's guide](https://guide.encode.moe/encoding/preparation.html#the-frameserver) 和 [VapourSynth documentation](http://www.vapoursynth.com/doc/index.html)。
因此，本指南不会涵盖安装和配置。

在开始编写脚本前，知道每个clip/filter都必须有一个变量名是非常重要的:

```py
clip_a = source(a)
clip_b = source(b)

filter_a = filter(clip_a)
filter_b = filter(clip_b)

filter_x_on_a = filter_x(clip_a)
filter_y_on_a = filter_y(clip_b)
```

此外，许多方法都在脚本模块或类似的集合中。
这些必须手动加载，然后在给定的别名下找到：

```py
import vapoursynth as vs
core = vs.core
import awsmfunc as awf
import kagefunc as kgf
from vsutil import *

bbmod = awf.bbmod(...)
grain = kgf.adaptive_grain(...)
deband = core.f3kdb.Deband(...)

change_depth = depth(...)
```

为避免方法名冲突，通常不建议这样做 `from x import *`。

虽然此类模块中有许多filters，但也有一些filters可用作插件。
这些插件可以通过 `core.namespace.plugin` 或 替代调用 `clip.namespace.plugin`。
这意味着以下两个是等效的：

```py
via_core = core.std.Crop(clip, ...)
via_clip = clip.std.Crop(...)
```

脚本里无法直接使用这些方法，这意味着以下是不可能的

```py
not_possible = clip.awf.bbmod(...)
```

在本指南中，我们将会对要使用的源进行命名并设置变量名为 `src` 以方便之后反复对其操作。
