# EE243 Final Project: A Qualitative Analysis of Wan

**Worked on by:** Muhammad Memon & Aamir Khan  
**Course:** EE 243, University of California, Riverside

**[Full Report (PDF)](EE243_Final_Project.pdf)**  

**Demo Video:** [YouTube](https://youtu.be/WfdSfl_S8Sk) | [Google Drive backup](https://drive.google.com/file/d/134bAyWQVPON9tZ8lCDbDwC6y0TW-KvmN/view?usp=share_link)

---

## Overview

This project is a **qualitative study of [Wan](https://arxiv.org/abs/2503.20314) - *Open and Advanced Large-Scale Video Generative Models***. It's Alibaba's family of open-weight video foundation models. Wan is built on a diffusion-transformer backbone paired with a video autoencoder (Wan-VAE) and a text encoder, and a single foundation model handles both **text-to-video (T2V)** and **image-to-video (I2V)** generation.

Instead of training or fine-tuning anything, we work **entirely within the released checkpoints** and probe them empirically to understand where the model genuinely succeeds and where it breaks down.

Because the larger Wan 2.2 checkpoints need many GBs of VRAM (beyond the free Colab tier), we target **Wan 2.1 T2V-1.3B** - the only checkpoint in the family that fits comfortably on free consumer-grade hardware (~8 GB, 480p). This makes **model scale an explicit variable** in our analysis: when a limitation appears, we ask whether it reflects Wan as a whole or simply the smallest, lowest-resolution tier of the open frontier.

## Objectives

1. **Contextualize Wan** relative to prior work - the Mixture-of-Experts architecture for video diffusion, the high-compression Wan2.2-VAE, and the large jump in training data over Wan 2.1.
2. **Showcase strengths** - qualitative success cases across text-to-image, text-to-video, and image-conditioned generation.
3. **Expose limitations** through targeted experiments, going deep on two in particular: **counting / numerical fidelity** and **compositional attribute binding**.
4. **Stay reproducible** - released checkpoints only, no training or fine-tuning, qualitative evidence.

## What We Did

### Strengths
- **Text-to-Image & Text-to-Video**: Generated a wide range of prompts, from photorealistic portraits to cinematic nature scenes. Output quality was frequently surprising (e.g. a "pack of wolves trotting through a snowy pine forest at dawn"), with results that looked straight out of a nature documentary.
- **Image-to-Image & Image-to-Video**: Tested Wan's conditioning claims and found them noticeably less reliable: cartoon inputs collapsed instead of animating, and real-photo inputs kept the background but altered the subjects.

### Limitations (the main focus)
- **The Counting Problem**: Using the template *"{N} red apple(s) on a wooden table…"* swept across N = 1-6 and multiple random seeds. Wan reliably handles low counts (N ≤ 2) but degrades sharply beyond that, and **never once produced 6 apples** across any seed, failing via overshoot, undershoot, over-saturation, and outright collapse to a single apple.
- **The Binding Problem**: Tested compositional attribute binding ("a red mug and a blue book" vs. its color-swapped twin). Wan binds **color** attributes correctly in most cases, even when a third object is added, but exhibits **seed-dependent color bleed and object merging** on the same seeds that destabilized counting. Relative-*size* binding could not be cleanly evaluated due to depth/framing confounds.

A key idea ties the two limitations together: the color drift seen during counting was **not** a general failure to follow modifiers (binding works fine), but rather a consequence of rendering **many identical instances**. Generation stability also appears partly tied to the **noise seed itself**.

## Repository Contents

| Path | Description |
| --- | --- |
| [`wan_t2i_t2v_strengths.ipynb`](wan_t2i_t2v_strengths.ipynb) | Text-to-image and text-to-video strength experiments |
| [`wan_i2i_i2v_strengths.ipynb`](wan_i2i_i2v_strengths.ipynb) | Image-to-image and image-to-video conditioning experiments |
| [`wan_t2v_limitations.ipynb`](wan_t2v_limitations.ipynb) | Counting and attribute-binding limitation experiments |
| [`Inputs/`](Inputs) | Reference images fed into the image-conditioned experiments (cartoon and movie clips) |
| [`generated/`](generated) | All model outputs, organized into `Strengths/` (T2I, T2V, I2I, I2V), `Weaknesses/` (counting fidelity & attribute binding sweeps), and `Figures/` (frames used in the report) |
| [`EE243_Final_Project.pdf`](EE243_Final_Project.pdf) | Full written report |

## Setup

Experiments were run on **Google Colab** using the **Wan 2.1 T2V-1.3B** checkpoint via the Hugging Face `diffusers` integration. Generation parameters were held fixed (480p, 81 frames, 30 inference steps, guidance scale ≈ 5.0, `flow_shift=3.0`) so that only the **prompt** and **random seed** varied across experiments.

## Conclusion

Wan 2.1 T2V-1.3B delivers the photorealistic quality the Wan line is known for, but intentional probing reveals clear bottlenecks: unreliable image-conditioned generation, a limit on counting, and imperfect attribute binding. Because every result here is specific to the smallest 1.3B variant at 480p, our natural next step would be to test whether Wan 2.2's MoE capacity and larger training corpus push these failure boundaries outward.
