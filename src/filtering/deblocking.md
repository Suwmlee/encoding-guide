Deblocking is mostly equivalent to smoothing the source, usually with
another mask on top. The most popular function here is `Deblock_QED`
from `havsfunc`. The main parameters are

-   `quant1`: Strength of block edge deblocking. Default is 24. You may
    want to raise this value significantly.

-   `quant2`: Strength of block internal deblocking. Default is 26.
    Again, raising this value may prove to be beneficial.

Other popular options are `deblock.Deblock`, which is quite strong, but
almost always works,\
`dfttest.DFTTest`, which is weaker, but still quite aggressive, and
`fvf.AutoDeblock`, which is quite useful for deblocking MPEG-2 sources
and can be applied on the entire video. Another popular method is to
simply deband, as deblocking and debanding are very similar. This is a
decent option for AVC Blu-ray sources.
