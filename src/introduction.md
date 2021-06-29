# 介绍

<https://git.concertos.live/Encode_Guide/mdbook-guide>

本指南既可以作为对制作高质量编码作品感兴趣的新手的起点，也可以作为有经验的压制人员的参考。
因此，在大多数功能介绍完并解释了它们的用途之后，可以找到深入的解释。
这些只是为好奇的人提供简单扩展，完全不是靠阅读来应用该功能。

# Terminology

For this to work, basic terms and knowledge about video is required.
These explanations will be heavily simplified, as there should be dozens of web pages with explanations on these subjects on sites like Wikipedia, the AviSynth wiki etc.

## Video

消费类视频产品通常存储在 YCbCr 中，该术语通常与 YUV 互换使用。在本指南中，我们将主要使用术语 YUV，因为 VapourSynth 格式编写也是 YUV。

YUV formatted content has information split into three planes: Y, referred to as luma, which represents the brightness, and U and V, which each represent a chroma plane.
These chroma planes represent offsets between colors, whereby the middle value of the plane is the neutral point.

This chroma information is usually subsampled, meaning it is stored in a lower frame size than the luma plane.
Almost all consumer video will be in 4:2:0, meaning chroma planes are half of the luma plane's size.
[The Wikipedia page on chroma subsampling](https://en.wikipedia.org/wiki/Chroma_subsampling) should hopefully suffice to explain how this works.
As we usually want to stay in 4:2:0 format, we are restricted in our frame dimensions, as the luma plane's size has to be divisible by 2.
This means we cannot do uneven cropping or resizing to uneven resolutions.
However, when necesssary, we can work in each plane individually, although this will be explained in the [filtering part]().

Additionally, our information has to be stored with a specific precision.
Usually, we deal with 8-bit per-plane precision.
However, for UHD Blu-rays, 10-bit per-plane precision is the standard.
This means possible values for each plane range from 0 to \\(2^8 - 1 = 255\\).
In [the bit depths chapter](filtering/bit_depths.md), we will introduce working in higher bit depth precision for increased accuracy during filtering.

## VapourSynth

For loading our clips, removing unwanted black borders, resizing, and combatting unwanted artifacts in our sources, we will employ the VapourSynth framework via Python.
While using Python might sound intimidating, those with no prior experience need not worry, as we will only be doing extremely basic things.

There are countless resources to setting up VapourSynth, e.g. [the Irrational Encoding Wizardry's guide](https://guide.encode.moe/encoding/preparation.html#the-frameserver) and the [VapourSynth documentation](http://www.vapoursynth.com/doc/index.html).
As such, this guide will not be covering installation and setup.

To start with writing scripts, it is important to know that every clip/filter must be given a variable name:

```py
clip_a = source(a)
clip_b = source(b)

filter_a = filter(clip_a)
filter_b = filter(clip_b)

filter_x_on_a = filter_x(clip_a)
filter_y_on_a = filter_y(clip_b)
```

Additionally, many functions are in script collections or similar.
These must be loaded manually and are then found under the given alias:

```py
import awsmfunc as awf
import kagefunc as kgf
from vsutil import *

bbmod = awf.bbmod(...)
grain = kgf.adaptive_grain(...)

change_depth = depth(...)
```

So as to avoid conflicting function names, it is usually not recommended to do `from x import *`.

While many filters are in such collections, there are also filters that are available as plugins.
These plugins can be called via `core.namespace.plugin` or alternatively `clip.namespace.plugin`.
This means the following two are equivalent:

```py
via_core = core.std.Crop(clip, ...)
via_clip = clip.std.Crop(...)
```

这对于脚本下的函数是不可能的，这意味着以下是不可能的

```py
not_possible = clip.awf.bbmod(...)
```

In this guide, we will name the source clip to be worked with `src` and set variable names to reflect what their operation does.
