Denoising is a rather touchy subject. Live action encoders will never
denoise, while anime encoders will often denoise too much. The main
reason you'll want to do this for anime is that it shouldn't have any on
its own, but compression introduces noise, and bit depth conversion
introduces dither. The former is unwanted, while the latter is wanted.
You might also encounter intentional grain in things like flashbacks.
Removing unwanted noise will aid in compression and kill some slight
dither/grain; this is useful for 10-bit, since smoother sources simply
encode better and get you great results, while 8-bit is nicer with more
grain to keep banding from appearing etc. However, sometimes, you might
encounter cases where you'll have to denoise/degrain for reasons other
than compression. For example, let's say you're encoding an anime movie
in which there's a flashback scene to one of the original anime
episodes. Anime movies are often 1080p productions, but most series
aren't. So, you might encounter an upscale with lots of 1080p grain on
it. In this case, you'll want to degrain, rescale, and merge the grain
back[^38]:

    degrained = core.knlm.KNLMeansCL(src, a=1, h=1.5, d=3, s=0, channels="Y", device_type="gpu", device_id=0)
    descaled = fvf.Debilinear(degrained, 1280, 720)
    upscaled = nnedi3_rpow2(descaled, rfactor=2).resize.Spline36(1920, 1080).std.Merge(src, [0,1])
    diff = core.std.MakeDiff(src, degrained, planes=[0])
    merged = core.std.MergeDiff(upscaled, diff, planes=[0])
