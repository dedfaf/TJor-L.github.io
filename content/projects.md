---
title: 'Experience'
date: 2023-10-24
type: landing

design:
  # Section spacing
  spacing: '5rem'

# Page sections
sections:
  - block: resume-projects
    content:
      title: Selected Projects
      text: I enjoy making things. Here are a selection of projects that I have worked on over the years.
      # Upload project images to your `assets/media/` folder and reference the filename in the `image` option
      items:
        - title: Mixed-View Panorama Synthesis
          description: Designed a diffusion model to synthesize panoramic street-view scenes by fusing satellite imagery, depth estimation, and texture information with Transformer backbones.
          image: panorama.png
          url: https://github.com/TJor-L/MixedViewDiff
        - title: Pose-Driven Video Generation with Dual ControlNet
          description: Created a dual-ControlNet diffusion pipeline to generate videos conditioned on human poses and reference actions. Integrated LoRA for fine-tuning control styles.
          image: controlnet.png
          url: https://github.com/TJor-L/SD-based-video-generator
        - title: Wings of Resistance – 6502 Assembly Game
          description: Developed a narrative-based 2D flight combat game for Atari using 6502 Assembly, featuring pixel-level rendering, radar mechanics, and dynamic effects.
          image: game.png
          url: https://github.com/TJor-L/Wing-of-Resistance
---
