The term \"ringing\" can refer to a lot of edge artifacts, with the most
common ones being mosquito noise and edge enhancement artifacts. Ringing
is something very common with low quality sources. However, due to poor
equipment and atrocious compression methods, even high bitrate concerts
are prone to this. To fix this, it's recommended to use something like
`HQDeringmod` or `EdgeCleaner` (from `scoll`), with the former being my
recommendation. The rough idea behind these is to blur and sharpen
edges, then merge via edgemasks. They're quite simple, so you can just
read through them yourself and you should get a decent idea of what
they'll do. As `rgvs.Repair` can be quite aggressive, I'd recommend
playing around with the repair values if you use one of these functions
and the defaults don't produce decent enough results.

![`HQDeringmod(mrad=5, msmooth=10, drrep=0)` on the right, source on the
left. This is *very* aggressive deringing that I usually wouldn't
recommend. The source image is from a One Ok Rock concert Blu-ray with
37 mbps video.](Pictures/dering.png){#fig:17}
