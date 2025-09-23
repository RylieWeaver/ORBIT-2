# ORBIT-2: Scaling Exascale Vision Foundation Models for Weather and Climate Downscaling
<p align="center"> <img src="docs/figs/example_downscale.png" width="540px"> </p> 
<p align="center"> <img src="docs/figs/example_downscale2.png" width="640px"> </p>

This repository contains code accompanying the paper [**ORBIT-2: Scaling Exascale Vision Foundation Models for Weather and Climate Downscaling**](https://arxiv.org/pdf/2505.04802).

## Overview
ORBIT-2 is a scalable foundation model for global, hyper-resolution climate downscaling. ORBIT-2 incorporates two key innovations:
 (1) Residual Slim ViT (Reslim), a lightweight architecture with residual learning and Bayesian regularization for efficient, robust prediction; and 
 (2) TILES, a tile-wise sequence scaling algorithm that reduces self-attention complexity from quadratic to linear, enabling long-sequence processing and massive parallelism. 
 ORBIT-2 scales to 10 billion parameters across 65,536 GPUs, achieving up to 4.1 ExaFLOPS sustained throughput and 74–98% strong scaling efficiency. It supports downscaling to 0.9 km global resolution and processes sequences up to 4.2 billion tokens. On 7 km resolution benchmarks, ORBIT-2 achieves high accuracy with 𝑅2 scores in range of 0.98–0.99 against observation data.

## Motivation
Conventional downscaling methods, such as dynamical approaches (nested climate models) and statistical approaches (regression-based mappings), are limited. Dynamical downscaling requires massive supercomputers and weeks of computation, while statistical methods often lack physical consistency and generalization across regions or variables. ORBIT-2 overcomes these challenges by combining physics-informed AI with exascale scalability, enabling global hyper-resolution downscaling in seconds rather than weeks. Once trained, ORBIT-2 runs efficiently even on modest hardware or edge devices, providing global downscaling in millisecond. In addition, ORBIT-2 provides a single, generalizable foundation model that delivers physically consistent, high-resolution climate predictions across diverse regions and variables—something conventional methods cannot achieve.

## Reslim Architecture
Reslim is a vision transformer (ViT) architecture that operates and trains directly on adaptively compressed spatial inputs, significantly reducing sequence length while preserving critical information. It preserves accuracy and reduces uncertainty through a lightweight residual learning architecture, enabling efficient, low-overhead predictions.


<p align="center">
  <img src="docs/figs/reslim.png" width="640px">
</p>



## TILES Sequence Scaling Algorithm
TILES is a ViT training algorithm that reduces ViT’s self-attention complexity from quadratic to linear. It works by dividing images into overlapping tiles, each processed in parallel on separate Graphical Process Units (GPUs) using localized self-attention. Each tile’s downscaled outputs are then seamlessly merged to the full image.

<p align="center">
  <img src="docs/figs/TILES.png" width="400px">
</p>


## Installation
**To Do: Isaac, please fill in here. Provide clear installation instructions. Specify all dependencies (requirements.txt or equivalent).

(1) Install your conda environment
(2) Then do pip install -e . to install the package to the environment
(3) Do  pip install -r requirements.txt



## Tutorial Example
**To Do: Hong-Jun, fill in here for frontier supercomputer example.
**Isaac, fill in here for the DGX box example. 


## Hyperparameter Configuration
**To Do: Isaac, fill in here.Explain how to use configuration files for experiments.

## Pretrained and Fine-Tuned Model Checkpoints
**To Do: Jong-Youl, can you upload the checkpoints and include the path here.**


## Pretraining and Fine-Tuning Datasets on Frontier
See Table 1 of the orbit-2 paper at [ORBIT-2 Paper on arXiv](https://arxiv.org/pdf/2505.04802) , for datasets trained for checkpoints

## Performance
### Strong Scaling on Frontier

<p align="center">
  <img src="docs/figs/strong_scaling.png" width="640px">
</p>

### Maximum Sequence Length Scaling


### Model Inference Speed
A key advantage of AI foundation models is their efficiency at inference. Once trained, they can be deployed on edge devices with limited resources and deliver near real-time predictions. We evaluate inference performance on 8 GPUs, using ERA5 to ERA5 downscaling from 112 km to 28 km. For the 9.5M-parameter model, downscaling each global sample requires only 4 millisecond. For the 10B-parameter model, it takes 0.55 second. In contrast, dynamic numerical downscaling approaches require days or weeks of computation on a large supercomputer. This highlights the unmatched prediction speed of AI, enabling deployment in resource-limited environments with near real-time performance.


### Climate Analysis
ORBIT-2 achieves high-fidelity precipitation downscaling (1998–2021) across 58 IPCC climate regions. Key metrics include R² correlation, SSIM, monsoon onset/withdrawal timing, precipitation seasonality, entropy, and Hovmöller diagnostics. The figure on the left shows ERA5 at 28 km resolution, exhibiting moderate skill scores across metrics and regions, whereas ORBIT-2 7 km downscaling (figure on the right) consistently achieves much higher skill scores across all metrics and monsoon regions. These results highlight the effectiveness of ORBIT-2 in enhancing the spatiotemporal fidelity of precipitation, especially in regions governed by complex climatic processes.
<p align="center"
  <img src="docs/figs/climate_analysis.png" width="640px">
</p>



### Animation
ERA5 input at 28km and ORBIT-2 downscaled output at 7km for global precipitation.
[ORBIT-2 animation](https://www.youtube.com/watch?v=Iahsl1L_1jQ) 
