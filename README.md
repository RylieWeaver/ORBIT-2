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
TILES is a ViT training algorithm that reduces ViT’s self-attention complexity from quadratic to linear. It works by dividing images into overlapping tiles, each processed in parallel on separate GPUs using localized self-attention. Each tile’s downscaled outputs are then seamlessly merged to the full image.

Key features:
- **Tile Division**: Images are split into div×div tiles (e.g., div=4 creates 16 tiles)
- **Overlapping Regions**: Adjacent tiles share overlap pixel rows/columns to ensure continuity
- **Patch Size Compatibility**: Tile dimensions must be divisible by the transformer's patch_size
- **Linear Complexity**: Enables processing of sequences up to 4.2 billion tokens

<p align="center">
  <img src="docs/figs/TILES.png" width="400px">
</p>


## Installation
**To Do: Isaac, please fill in here. Provide clear installation instructions. Specify all dependencies (requirements.txt or equivalent).

(1) Install your conda environment
(2) Then do pip install -e . to install the package to the environment
(3) Do  pip install -r requirements.txt



## Tutorial Example

### Running on Frontier Supercomputer

#### Prerequisites
- Access to Frontier supercomputer at ORNL
- Allocated compute hours (replace 'lrn036' with your project allocation)
- Conda environment setup (see Installation section above)

#### Step 1: Configure Your Experiment
Choose an appropriate configuration file from `configs/`:
- `interm_8m.yaml`: 8M parameter model for testing
- `interm_117m.yaml`: 117M parameter model
- `interm_1b.yaml`: 1B parameter model
- `interm_10b.yaml`: 10B parameter model (requires more nodes)

Key parameters to modify:

**Model Configuration:**
```yaml
model:
  preset: res_slimvit  # Model architecture: 'res_slimvit' (ORBIT-2) or 'vit' (standard ViT)
  lr: 2e-3             # Learning rate
  superres_mag: 4      # Super-resolution magnification factor
  patch_size: 2        # Patch size for vision transformer
  embed_dim: 256       # Embedding dimension (increase for larger models)
  depth: 6             # Number of transformer blocks
  decoder_depth: 4     # Number of decoder layers
  num_heads: 4         # Number of attention heads
```

**Data Configuration:**
```yaml
data:
  # Select datasets by uncommenting desired options
  low_res_dir: {
    # ERA5 global reanalysis data
    #'ERA5_1': "/path/to/era5/5.625_deg/",  # ~625km resolution
    #'ERA5_2': "/path/to/era5/1.0_deg/",    # ~111km resolution
    
    # PRISM regional climate data (CONUS)
    'PRISM': "/path/to/prism/10.0_arcmin",   # ~18km resolution
    
    # DAYMET regional climate data (North America)
    #'DAYMET_1': "/path/to/daymet/10.0_arcmin",  # ~18km resolution
    #'DAYMET_2': "/path/to/daymet/2.0_arcmin",   # ~4km resolution
  }
  
  high_res_dir: {
    # Target high-resolution data
    #'ERA5_1': "/path/to/era5/1.40625_deg/",     # ~156km resolution
    #'ERA5_2': "/path/to/era5/0.25_deg/",        # ~28km resolution
    'PRISM': "/path/to/prism/2.5_arcmin",        # ~4.5km resolution
    #'DAYMET_1': "/path/to/daymet/2.5_arcmin",   # ~4.5km resolution
    #'DAYMET_2': "/path/to/daymet/0.5_arcmin",   # ~0.9km resolution
  }

  # Variables to use for training (modify based on dataset)
  dict_out_variables: {
    'PRISM': [
      "total_precipitation_24hr",  # Daily precipitation
      "2m_temperature_min",        # Daily minimum temperature
      "2m_temperature_max",        # Daily maximum temperature
    ],
    'ERA5_1': [
      "total_precipitation_24hr",
      "2m_temperature_min",
      "2m_temperature_max",
      #"10m_u_component_of_wind",  # Uncomment for wind variables
      #"10m_v_component_of_wind",
    ],
  }
```

**Training Configuration:**
```yaml
trainer:
  max_epochs: 100      # Training epochs
  batch_size: 32       # Batch size per GPU
  data_type: bfloat16  # Use bfloat16 for memory efficiency
  train_loss: "bayesian_tv"  # Loss function: 'mse', 'bayesian_tv', 'perceptual'

parallelism:
  fsdp: 4              # Fully Sharded Data Parallel
  tensor_par: 1        # Tensor parallelism
  seq_par: 1           # Sequence parallelism

# TILES: Tile-wise sequence scaling algorithm
tiling:
  do_tiling: False     # Enable TILES for processing very large images/sequences
  div: 4               # Division factor: splits image into div×div tiles
                       # div=2 → 4 tiles (2×2), div=4 → 16 tiles (4×4)
  overlap: 3           # Number of pixel rows/columns to overlap between adjacent tiles
                       # Important: Total tile size must be divisible by patch_size
                       # If not, you'll get an error asking to adjust overlap
                       # Error message will tell you how many pixels to add
```

#### Step 2: Submit Training Job
First, edit `launch_intermediate.sh` to update:
- `#SBATCH -A lrn036` → your project allocation
- `conda activate` path → your conda environment
- `../configs/interm_8m.yaml` → your chosen config file (e.g., `interm_117m.yaml`, `interm_1b.yaml`)

Then submit the job:
```bash
cd examples/
sbatch launch_intermediate.sh
```

The script will:
- Load required modules (ROCm 6.3.1, libfabric, aws-ofi-rccl)
- Set up environment variables for optimal AMD GPU performance
- Launch training with 8 GPUs per node
- Output logs to `flash-{JOBID}.out`

#### Step 3: Monitor Training
```bash
# Check job status
squeue -u $USER

# Monitor training progress
tail -f flash-{JOBID}.out
```

#### Step 4: Visualize Results
After training completes or reaches a checkpoint, edit `launch_visualize.sh` to update:
- `#SBATCH -A lrn036` → your project allocation
- `conda activate` path → your conda environment
- `../configs/interm_8m_ft.yaml` → your config file

For visualization, ensure your config file specifies:
```yaml
trainer:
  pretrain: /path/to/your/checkpoint.pth  # Path to trained model checkpoint
```

Then submit the visualization job:
```bash
sbatch launch_visualize.sh
```

This will generate visualization outputs for:
- Input low-resolution data
- ORBIT-2 downscaled predictions at high resolution
- Comparison metrics

Note: Currently visualizes test sample at index=0. Future updates will allow custom sample selection.

#### Example Output
Training on 1 node (8 GPUs) with the 8M model typically:
- Processes ~400 samples per epoch
- Achieves ~X samples/second throughput
- Reaches R² > 0.98 after ~50 epochs

#### Scaling to Multiple Nodes
For larger models or datasets, modify the SLURM parameters:
```bash
#SBATCH --nodes=8          # Use 8 nodes (64 GPUs)
#SBATCH -t 02:00:00        # Extend time limit
```

And enable advanced parallelism in config:
```yaml
parallelism:
  fsdp: 8
  tensor_par: 2
  seq_par: 2
```

#### Troubleshooting
- If encountering OOM errors, reduce `batch_size` or enable gradient checkpointing
- For network issues, check the NCCL environment variables
- Ensure DDStore is properly configured for checkpointing

### Running on DGX Systems
**To Do: Isaac, fill in here for the DGX box example. 


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
