# Neural Rendering — Presentation Script (English)

> **Duration:** ~15–20 minutes (excluding Q&A)
> **Structure:** Slide-by-slide script + key explanation points
> **Tip:** Press `S` on the slide to show brief speaker notes.

---

## Slide 1 — Cover

Hello everyone, my name is Sieun Kim.
Today I'll be talking about **Neural Rendering** —
how we can use neural networks to reconstruct and synthesize 3D scenes from photographs.

Quick question before we start:
has anyone here used an app like Luma AI, or tried the spatial scanning feature on an iPhone?
The technology behind those experiences is exactly what we're going to cover today.

Here's the roadmap:
we'll start by looking at why classical 3D reconstruction hits a wall,
then walk through photogrammetry as the stepping stone,
and then dive into two modern neural approaches — NeRF and 3D Gaussian Splatting.

---

## Slide 2 — Why Classical Methods Fall Short

There are two dominant approaches to classical 3D reconstruction.

The first is **manual 3D modeling** — artists building scenes by hand in tools like Blender or Maya.
This gives full creative control, but even a simple object can take dozens of hours.
And there's simply no way to faithfully reproduce the real world at scale.

The second is **photogrammetry** — automatically reconstructing 3D from photos.
It works well for certain things, but it completely falls apart on transparent objects,
reflective surfaces, and thin structures like wire or hair.

Both methods share the same fundamental limitation:
**neither can produce a photorealistic rendering from an arbitrary new viewpoint.**
That is exactly the problem Neural Rendering is designed to solve.

---

## Slide 3 — How Photogrammetry Works

Before we get to neural approaches, let's briefly understand photogrammetry —
because it's actually the starting point for NeRF as well.

The idea is this: take many photos from different angles,
find the **same physical point** across multiple images,
and triangulate its 3D position.
This process is called **Structure from Motion**, or SfM.

The output is a **point cloud** — millions of 3D points.
It works beautifully on buildings, statues, rocky terrain —
anything that's opaque, matte, and texture-rich.

It struggles with glass, mirrors, fur, and anything whose appearance
changes depending on the viewing angle — which is most of the interesting stuff.

The standard open-source tool for this is **COLMAP**,
and notably, NeRF uses COLMAP as a preprocessing step to figure out where the cameras were.

---

## Slide 4 — What is Neural Rendering?

So what does Neural Rendering do differently?

The core idea is this:
**instead of explicitly storing geometry, a neural network learns to represent the scene itself.**

Classical methods ask: "what does this surface look like?"
Neural Rendering asks: "if I stand at position X looking in direction D, what color do I see?"
The network learns to answer that question for any position and direction —
including ones that were never photographed.

There are two key methods today.

**NeRF** — Neural Radiance Field, 2020.
A small MLP network implicitly encodes the entire scene.
Keywords: Implicit · MLP · Volume Rendering.

**3D Gaussian Splatting** — 3DGS, 2023.
Millions of explicit 3D ellipsoids represent the scene.
Keywords: Explicit · Rasterization · Real-time.

Both enable **novel view synthesis** — you can place a virtual camera anywhere and get a photorealistic render.

---

## Slide 5 — How NeRF Works

Let me walk you through the NeRF pipeline step by step.

**① Input: 50–200 photos + camera poses.**
COLMAP estimates where each photo was taken in 3D space.
Each photo becomes a training example.

**② Cast a ray through each pixel.**
For every pixel we want to render, we shoot a ray from the camera
out into the 3D scene. This is the reverse of how light actually travels —
we're tracing the path light would have taken to reach that pixel.

**③ Sample points along the ray and query the MLP.**
We place 64–128 sample points along the ray,
and for each point we feed five numbers into the network:
position (x, y, z) and viewing direction (θ, φ).
The network outputs two things: **RGB color** and **σ, density**.
High density means there's something there. Low density means empty space.

**④ Volume rendering produces the pixel color.**
We composite the samples front to back.
High-density points occlude what's behind them; low-density points are transparent.
The result is a single pixel color.

**⑤ Compare against the real photo and update the network.**
We compute the difference between the predicted and real pixel color,
and backpropagate to update the MLP weights.
Repeat this millions of times across all rays in all training images.
Training takes 1–8 hours on a single GPU.

---

## Slide 5b — NeRF Result

Let's look at what this actually produces.

What you see here is the **lego bulldozer** scene —
one of the most iconic results from the original NeRF paper.
The model was trained on about 100 photos of a physical lego set.

There is no 3D model file. No artist touched this.
The neural network inferred the geometry and appearance entirely from photographs.
The camera angle you're seeing was **never physically photographed**.

On the upper right you can see the instant-ngp interface —
that's NVIDIA's open-source implementation.
This quality is achievable on a laptop GPU in a matter of minutes.

---

## Slide 6 — How 3D Gaussian Splatting Works

3DGS takes a fundamentally different approach from NeRF.
Instead of a neural network, it uses millions of **3D Gaussians** — small ellipsoids — to represent the scene.

The pipeline:

**① Start from the SfM point cloud.**
Take the sparse 3D points from COLMAP and use them as seeds for Gaussians.

**② Each point becomes a 3D Gaussian.**
Each Gaussian has four learnable parameters:
- **μ** — position (x, y, z)
- **Σ** — shape and orientation (how elongated and in what direction)
- **SH** — color, encoded as spherical harmonics so it can change with viewing angle
- **α** — opacity

**③ Project each Gaussian onto the image plane — this is the "splatting".**
From the current camera's perspective, each 3D Gaussian becomes a 2D ellipse.

**④ Sort by depth and alpha-composite front to back.**
Blend all the 2D ellipses in depth order to produce the final pixel colors.

Because there's no neural network at render time —
we're just rasterizing geometric primitives —
this achieves **30–100+ fps in real time** after training.
Training itself is also much faster: roughly 30 minutes versus hours for NeRF.

---

## Slide 6b — 3DGS Result

This is the **bicycle scene** from the original 3DGS paper —
a real outdoor scene captured with a camera and reconstructed as millions of 3D Gaussians.

Look at the individual blades of grass, the spokes of the wheel, the leaves in the trees.
All of that detail is represented by tiny 3D ellipsoids learned from photos.

Unlike NeRF, this scene renders in **real time**.
The speed is comparable to what you'd need for a game engine.

This image is from Kerbl et al., SIGGRAPH 2023, published by INRIA.

---

## Slide 7 — Results & Real-world Examples

Let's look at where this technology shows up in the real world.

**NeRF Studio** — an open-source framework from UC Berkeley.
It packages many NeRF variants into one easy-to-use system:
Nerfacto, Instant-NGP, Splatfacto, and more.

**Luma AI** — a consumer iPhone app.
You walk around an object with your phone, upload the video,
and get a high-quality NeRF in minutes.
Already used in product photography and real-estate.

**3DGS (Kerbl et al.)** — the original paper published at SIGGRAPH 2023.
Open-source code on GitHub; it became a standard benchmark almost immediately.

The before/after comparison summarizes everything:
classical photogrammetry gives a sparse, holey point cloud that can't render novel views.
Neural rendering gives a dense, photorealistic output from any camera position.

---

## Slide 8 — Pros & Cons

Let me be clear about the strengths and limitations.

**Strengths:**
- **Photorealistic quality** — perceptual metrics (PSNR, SSIM) match or exceed traditional rendering.
- **Fully automatic** — all you need is photos. No manual modeling at all.
- **Novel view synthesis** — see viewpoints that were never physically captured.
- **Compact storage** — an entire NeRF scene fits in ~5MB of MLP weights.

**Limitations:**
- **Training time** — NeRF takes 1–8 hours; 3DGS takes ~30 minutes. Neither is instant.
- **Real-time rendering** — only 3DGS achieves this; vanilla NeRF renders at roughly 1 fps.
- **Scene editing is hard** — changing an object's color, removing it, or animating it
  requires specialized techniques far beyond vanilla training.
  You don't get a clean mesh you can modify freely.
- **Static scene assumption** — moving people or vehicles during capture create artifacts.

---

## Slide 9 — Current Limits & What's Next

Three open challenges remain.

**Dynamic scenes** —
Both NeRF and 3DGS assume the world is completely frozen during capture.
A person walking through the scene breaks the reconstruction.
Active research: D-NeRF, 4D Gaussians, and deformable Gaussian methods.

**Large-scale scenes** —
Reconstructing a city block requires tiling the scene and distributed training.
The math doesn't scale directly to unbounded outdoor environments.

**Relighting** —
The color learned by the network bakes in the original lighting conditions.
Changing the light direction or placing the scene in a new environment
requires explicit material decomposition — separating albedo from lighting.

**What's coming:**
- Real-time NeRF on mobile phones (Instant-NGP variants)
- Text → 3D: DreamFusion, Shap-E — no photos required, just a text prompt
- Integration with Unreal Engine and Unity — scan the real world, render in a game engine at 60fps
- Applications in surgical planning, robot navigation, and cultural heritage preservation

---

## Closing

To summarize:

Classical 3D reconstruction — even photogrammetry — couldn't produce photorealistic novel views.
NeRF solved this for the first time in 2020 using a neural network as an implicit scene representation.
3DGS pushed it further in 2023 by enabling real-time rendering.

Dynamic scenes, scene editing, and large-scale capture are still open problems —
but this field has been moving faster than almost any other in computer graphics.
Every year since 2020 has brought a major step forward.

I'm happy to take any questions. Thank you.

---

*Slides: https://hundredman.github.io/cg-neural-rendering/*
