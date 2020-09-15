# Descaling

If you've read a bit about anime encoding, you've probably heard the term "descaling" before; this is the process of "reversing" an upscale by finding the native resolution and resize kernel used.
When done correctly, this is a near-lossless process and produces a sharper output than standard spline36 resizing with less haloing artifacts.
However, when done incorrectly, this will only add to the already existing issues that come with upscaling, such as haloing, ringing etc.

The most commonly used plugin to reverse upscales is [Descale](https://github.com/Irrational-Encoding-Wizardry/vapoursynth-descale), which is most easily called via [`fvsfunc`](https://github.com/Irrational-Encoding-Wizardry/fvsfunc), which has an alias for each kernel, e.g. `fvf.Debilinear`.
This supports bicubic, bilinear, lanczos, and spline upscales.

Most digitally produced anime content, especially TV shows, will be a bilinear or bicubic upscale from 720p, 810p, 864p, 900p, or anything in-between.
While not something that can only be done with anime, it is far more prevalent with such content, so we will focus on anime accordingly.
As our example, we'll look at Nichijou.
This is a bicubic upscale from 720p.
To check this, we compare the input frame with a descale upscaled back with the same kernel:

```py
b, c = 1/3, 1/3
descale = fvf.Debicubic(src, 1280, 720, b=b, c=c)
rescale = descale.resize.Bicubic(src, src.width, src.height, filter_param_a=b, filter_param_b=c)
merge_chroma = rescale.std.Merge(src, [0, 1])
out = core.std.Interleave([src, merge_chroma])
```

Here, we've merged the chroma from the source with our rescale, as chroma is a lower resolution than the source resolution, so we can't descale it.
The result:


<p align="center"> 
<img src='Pictures/resize0.png' onmouseover="this.src='Pictures/resize1.png';" onmouseout="this.src='Pictures/resize0.png';" />
</p>

As you can see, lineart is practically identical and no extra haloing or aliasing was introduced.

On the other hand, if we try an incorrect kernel and resolution, we see lots more artifacts in the rescaled image:

```py
descale = fvf.Debilinear(src, 1280, 720)
rescale = descale.resize.Bilinear(src, src.width, src.height)
merge_chroma = rescale.std.Merge(src, [0, 1])
out = core.std.Interleave([src, merge_chroma])
```

<p align="center"> 
<img src='Pictures/resize3.png' onmouseover="this.src='Pictures/resize4.png';" onmouseout="this.src='Pictures/resize3.png';" />
</p>

## Native resolutions and kernels

Now, the first thing you need to do when you want to descale is figure out what was used to resize the video and from which resolution the resize was done.
The most popular tool for this is [getnative](https://github.com/Infiziert90/getnative), which allows you to feed it an image, which it will then descale, resize, and calculate the difference from the source, then plot the result so you can find the native resolution.

For this to work best, you'll want to find a bright frame with very little blurring, VFX, grain etc.

Once you've found one, you can run the script as follows:

```sh
python getnative.py image.png -k bicubic -b 1/3 -c 1/3
```

This will output a graph in a `Results` directory and guess the resolution.
It's based to take a look at the graph yourself, though.
In our example, these are the correct parameters, so we get the following:

However, we can also test other kernels:

```sh
python getnative.py image.png -k bilinear
```

The graph then looks as follows:

If you'd like to test all likely kernels, you can use `--mode "all"`.

# Mixed Resolutions

## Chroma shifting and 4:4:4

# Upscaling and Rescaling

## Upscaling

## Rescaling

