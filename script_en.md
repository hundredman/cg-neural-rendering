# Neural Rendering — Presentation Script (English)

> **Duration:** ~15–20 minutes (excluding Q&A)
> **[brackets]** = action / pause cues
> *italics* = emphasize when speaking

---

## Slide 1 — Cover

Hi everyone.

So today I'll be talking about Neural Rendering.

Now, that term can sound pretty abstract, so let me give you the one-sentence version first.

You take a bunch of photos of a place.
A neural network looks at those photos and *learns* the scene.
And then — you can view it from angles the camera never actually went to.

Has anyone here used Luma AI ("loo-ma ay-eye")?
It's this iPhone app where you walk around an object,
and it reconstructs it in 3D.
That's exactly the kind of thing we're talking about today.

Here's the plan:
first we'll look at why old methods hit a wall,
then I'll walk you through two modern approaches — NeRF ("nerf") and 3D Gaussian Splatting ("three-dee gaussian splatting").

Let's go.

---

## Slide 2 — Why Classical Methods Fall Short

So how did people make 3D scenes before?

There are basically two ways.

One — artists build it by hand.
Tools like Blender or Maya, modeling everything manually.
Even a simple object takes dozens of hours.
Reproducing the real world at any real scale? Not happening.

Two — photogrammetry.
You take lots of photos and the software reconstructs 3D automatically.
Works pretty well for buildings and statues.
But the moment you point it at glass, or a mirror, or anything thin like wire or hair —
it completely falls apart.

Both approaches have the same core problem.

*"You can't produce a photorealistic image from a new viewpoint."*

That's exactly what Neural Rendering is trying to fix.

---

## Slide 3 — How Photogrammetry Works

Before we get into NeRF, let's quickly understand photogrammetry —
because NeRF actually builds on top of it.

The idea is pretty simple.

You take photos from a bunch of different angles.
You find the *same physical point* across multiple images.
And you triangulate its 3D position.

This is called Structure from Motion, or SfM ("ess-eff-em").

The output is a point cloud — millions of 3D dots floating in space.

For buildings, rock faces, statues — it looks great.
But for glass, mirrors, fur, anything that looks *different* depending on the angle —
it falls apart.

The standard open-source tool is COLMAP ("col-map").
And notably — NeRF uses COLMAP as a first step to figure out where the cameras were.

---

## Slide 4 — What is Neural Rendering?

So what does Neural Rendering do differently?

Old methods *store* geometry.
"This surface is here, and it's this color." Done.

Neural Rendering asks a different question:
*"If I'm standing at this position, looking in this direction — what color do I see?"*

A neural network learns to answer that question for any position and direction —
including ones no camera ever visited.

Right now there are two main methods.

**NeRF** — came out in 2020.
A tiny neural network implicitly encodes the entire scene inside its weights.
Keywords: Implicit, MLP ("em-el-pee"), Volume Rendering.

**3D Gaussian Splatting, 3DGS ("three-dee-gee-ess")** — 2023.
Millions of 3D ellipsoids explicitly represent the scene.
And it renders in *real time*.
Keywords: Explicit, Rasterization ("rass-ter-ih-ZAY-shun"), Real-time.

---

## Slide 5 — How NeRF Works

Okay, let me walk you through NeRF step by step.

[point to the pipeline diagram, left to right]

**Step one — photos in.**
50 to 200 photos, plus camera positions estimated by COLMAP.
Each photo is a training example.

**Step two — shoot a ray.**
For every pixel we want to render, we shoot a ray out from the camera into the scene.
Think of it as tracing the path light *would have taken* to reach that pixel, but in reverse.

**Step three — ask the network.**
We sample 64 to 128 points along that ray.
For each point, we feed five numbers into the network:
position x ("ex"), y ("why"), z ("zee"),
and viewing direction θ ("theta"), φ ("phi").
The network spits out two things:
RGB ("ar-gee-bee") color, and σ ("sigma") — density.
High σ (sigma) means something's there. Low σ (sigma) means empty space.

**Step four — volume rendering.**
We blend those samples front to back.
Dense points block what's behind them.
Transparent points let light through.
Result: one pixel color.

**Step five — compare and repeat.**
We compare the predicted color against the actual photo.
That difference is the loss.
Backpropagate, update the weights, repeat — millions of times.
Training takes one to eight hours on a single GPU ("gee-pee-you").

---

## Slide 5b — NeRF Result

Let's see what this actually looks like.

[pause, let the image land]

This is the lego bulldozer scene.

Someone photographed a physical lego set from about 100 angles.
Fed those photos into NeRF.
That's it.

No 3D model file. No artist. Nothing manual.
The network figured out the geometry and appearance entirely from photos.

And the viewpoint you're looking at right now?
*That angle was never photographed.*

The UI ("you-eye") on the top right is instant-ngp ("instant-en-gee-pee") —
NVIDIA's open-source implementation.
This quality, on a laptop GPU, in a few minutes.

---

## Slide 6 — How 3D Gaussian Splatting Works

3DGS takes a different angle — no pun intended.

Where NeRF hides everything inside a neural network,
3DGS uses millions of explicit little 3D blobs — Gaussians — to represent the scene directly.

[point to the pipeline, left to right]

**Step one — start from the point cloud.**
Take the sparse 3D points from COLMAP, use them as starting seeds.

**Step two — turn each point into a 3D Gaussian.**
Each Gaussian carries four things:
- μ ("mu") — position
- Σ ("capital sigma") — shape and orientation
- SH ("ess-aitch") — color, as spherical harmonics, so it changes with viewing angle
- α ("alpha") — opacity

**Step three — splatting.**
From the camera's perspective, each 3D Gaussian gets projected down to a 2D ellipse.

**Step four — sort and blend.**
Sort by depth, alpha ("alpha") composite front to back, done.

Here's the key thing:
*no neural network* at render time.
It's just rasterizing geometry.
So you get 30 to 100+ fps ("frames per second") in real time.
Training takes about 30 minutes.

---

## Slide 6b — 3DGS Result

Here's what 3DGS produces on a real outdoor scene.

[pause]

This is a bicycle in a park.

Look at the individual blades of grass.
The spokes.
The leaves.

All of that is millions of tiny 3D ellipsoids, learned from photos.

And unlike NeRF — this runs *in real time*.
We're talking game-engine speed.

This is from the original paper at SIGGRAPH ("sig-graph") 2023, by Kerbl et al. ("kerbl and colleagues") at INRIA ("in-ria").

---

## Slide 7 — Results & Real-world Examples

So where does this stuff actually show up?

**NeRF Studio** — open source, from UC Berkeley ("you-see berkeley").
Bundles a bunch of NeRF variants into one easy framework.
If you want to try training your own NeRF, this is probably where you'd start.

**Luma AI** — consumer iPhone app.
Walk around something with your phone, upload the video,
get a NeRF in minutes.
Already being used for product photography and real-estate listings.

**3DGS — Kerbl et al.** — SIGGRAPH 2023, code on GitHub ("git-hub").
Became a standard benchmark almost immediately after release.

And the before → ("arrow") after at the bottom tells the whole story:
on the left, a holey point cloud that you can't really render from new angles.
On the right, photorealistic output from anywhere.

---

## Slide 8 — Pros & Cons

Let me be honest about the trade-offs.

**What's great:**

The quality is genuinely photorealistic — perceptual metrics like PSNR ("pee-ess-en-ar") and SSIM ("ess-sim") are excellent.
It's fully automatic — just photos.
And NeRF packs an entire scene into ~("approximately") 5MB ("five megabytes") of network weights. That's tiny.

**What's not so great:**

Training time. NeRF takes one to eight hours. 3DGS is ~("approximately") 30 minutes. Neither is instant.

Real-time rendering — only 3DGS gets there. Vanilla NeRF is more like one fps ("one frame per second").

*Editing is really hard.*
Want to change an object's color? Remove something? Animate it?
You can't just grab a mesh and move vertices around.
You need specialized techniques on top of the basic training.

And both methods assume a *static scene*.
If someone walks through the frame while you're capturing — the reconstruction breaks.

---

## Slide 9 — Current Limits & What's Next

Three big open problems.

**One — dynamic scenes.**
Both methods assume the world is frozen while you're capturing.
Someone walks through? Reconstruction is ruined.
Active research: D-NeRF ("dee-nerf"), 4D Gaussians ("four-dee gaussians").

**Two — large scale.**
A single city block breaks the math.
You have to tile the scene and do distributed training.
Not easy.

**Three — relighting.**
The color the network learns has the original lighting *baked in*.
Move the scene to a different lighting environment?
You need to separately decompose albedo ("al-bee-do") from lighting — which is hard.

**What's coming:**

Real-time NeRF on phones.
Text → ("arrow") 3D with DreamFusion ("dream-fusion") and Shap-E ("shape-ee") — just describe what you want.
3DGS plugins for Unreal Engine ("un-real engine") and Unity ("you-nity").
And applications in surgery, robotics, and cultural preservation.

---

## Closing

So to wrap up.

Photogrammetry got us a long way — but it couldn't give us photorealistic new viewpoints.

NeRF solved that in 2020, using a neural network as an implicit scene representation.

3DGS took it further in 2023, making real-time rendering possible.

There's still work to do — dynamic scenes, editing, scale —
but this field has been moving *fast*.
Every year since 2020 has brought something that felt impossible the year before.

That's it from me. Happy to take questions. Thanks.

---

*Slides: https://hundredman.github.io/cg-neural-rendering/*
