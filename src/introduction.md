# Introduction

<https://git.concertos.live/Encode_Guide/mdbook-guide>

This guide is meant to be both a starting point for newcomers interested in producing high quality encodes as well as a reference for experienced encoders.
As such, after most functions are introduced and their uses explained, an in-depth explanation can be found.
These are meant as simple add-ons for the curious and should in no way be necessary reading to apply the function.

# Terminology

For this to work, basic terms and knowledge about video is required.
These explanations will be heavily simplified, as there should be dozens of web pages with explanations on these subjects on sites like Wikipedia, the AviSynth wiki etc.

## Video

Consumer video products are usually stored in YCbCr, a term which is commonly used interchangeably with YUV.
In this guide, we will mostly be using the term YUV, as VapourSynth formats are written as YUV.

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

TODO

- Make sure to remember to explain `core.namespace.plugin` and `videonode.namespace.plugin` are both possible!
