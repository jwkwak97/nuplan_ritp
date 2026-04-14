# nuplan-ritp (RITP · NVIDIA B200 Adaptation)

> **This repository is a fork of [Zigned/nuplan_zigned](https://github.com/Zigned/nuplan_zigned)**, adapted to run on **NVIDIA B200 GPUs** with an updated software stack (CUDA 12.4, PyTorch 2.5.1). The original code targets older hardware and package versions; this fork applies the following changes:
>
> - Upgraded to **Python 3.11**, **CUDA 12.4**, **PyTorch 2.5.1+cu124**
> - Fixed **pytorch-lightning 1.3.8** API compatibility (`pl.plugins.DDPPlugin` instead of removed `pl.strategies.DDPStrategy`)
> - Replaced removed `pytorch_lightning.metrics` with `torchmetrics`
> - Removed `torch.jit._overload` decorators incompatible with newer PyTorch
> - Fixed `os.getlogin()` crash in container/headless environments
> - Added nuplan-devkit **editable install** requirement (yaml configs missing from built package)
> - Updated `requirements.txt` / `requirements_torch.txt` / `setup.py` for current versions

Author's PyTorch implementation of [Reinforced Imitative Trajectory Planning for Urban Automated Driving](http://arxiv.org/abs/2410.15607).

## Getting Started

### 1. Installation

**Tested environment:** NVIDIA B200, Python 3.11, CUDA 12.4, PyTorch 2.5.1, pytorch-lightning 1.3.8

#### 1.1 Clone repositories

```bash
git clone https://github.com/motional/nuplan-devkit.git
git clone https://github.com/Zigned/nuplan_zigned.git
```

#### 1.2 Create conda environment

```bash
conda create -n nuplan_ritp python=3.11 -y
conda activate nuplan_ritp
```

#### 1.3 Install nuplan-devkit as editable

Install the devkit in editable mode so that Hydra config files (yaml) are accessible at runtime:

```bash
pip install -e ./nuplan-devkit --no-deps
```

> **Note:** Do **not** use a regular `pip install` for nuplan-devkit — the yaml config files are not included in the built package and Hydra will fail to find them.

#### 1.4 Install PyTorch and PyG dependencies

```bash
pip install -r nuplan_zigned/requirements_torch.txt
```

#### 1.5 Install nuplan_zigned dependencies

```bash
pip install -r nuplan_zigned/requirements.txt
```

#### 1.6 Install nuplan_zigned as editable

```bash
cd nuplan_zigned && pip install -e .
```

#### 1.7 Configure data paths in training scripts

Before running any training or caching script, open the script (e.g. `scripts/training/train_ritp_planner.py`) and set the following variables to match your machine:

```python
# Root directory containing nuplan-maps-v1.0/ and data/
NUPLAN_ROOT = '/path/to/your/nuplan/data'

# Parent directories of .db split folders
NUPLAN_DB_ROOT = '/path/to/your/nuplan/data/cache'

# Output directory for experiments and cache
SAVE_DIR = '/path/to/your/experiments/ritp_planner'

# (Optional) Pretrained model checkpoints — skipped automatically if files don't exist
QCMAE_DIR = '/path/to/qcmae.ckpt'
REWARDFORMER_DIR = '/path/to/rewardformer.ckpt'
U_R_STATS_DIR = '/path/to/u_r_stats.npy'
```

Expected data directory structure:

```
NUPLAN_ROOT/
├── nuplan-maps-v1.0/       # map files
└── data/
    └── cache/
        ├── train_boston/   # *.db files
        ├── train_pittsburgh/
        ├── train_singapore/
        └── val/
```



### 2. Training

#### 2.1 Training Bayesian RewardFormer

- Set `PYFUNC` to `'cache'` or `'train'` in `train_avrl_vector_rewardformer.py`, then run:

  ```bash
  python ~/nuplan_zigned/scripts/training/train_avrl_vector_rewardformer.py
  ```

  to cache features or train Bayesian RewardFormer, respectively.

- Run:

  ```bash
  python ~/nuplan_zigned/scripts/training/estimate_avrl_vector_rewardformer.py
  ```

  to estimate the mean and variance of the uncertainty of Bayesian RewardFormer following the AVRL method.

#### 2.2 Training QCMAE

- Set `PYFUNC` to `'cache'`, `'pretrain'`, or `'finetune'` in `train_qcmae.py`, then run:

  ```bash
  python ~/nuplan_zigned/scripts/training/train_qcmae.py
  ```

  to cache features, pretrain QCMAE, or finetune QCMAE, respectively.

#### 2.3 Training MotionFormer

- Set `PYFUNC` to `'cache'` or `'train'` in `train_ritp_planner.py`, then run:

  ```bash
  python ~/nuplan_zigned/scripts/training/train_ritp_planner.py
  ```

  to cache features or train MotionFormer, respectively.

### 3. Evaluation

- Run:

  ```bash
  python ~/nuplan_zigned/scripts/simulation/simulate_ritp_planner.py
  ```

  to conduct closed-loop simulation.

### 4. Visualization

- Run:

  ```bash
  python ~/nuplan_zigned/scripts/visualization/launch_nuboard_customized.py
  ```

  to launch our customized nuboard, where the simulated and ground truth trajectories of the ego vehicle are plotted.

## Citation

If you find this work useful, please cite:

```bibtex
@misc{zeng2025reinforcedimitativetrajectoryplanning,
  title        = {Reinforced Imitative Trajectory Planning for Urban Automated Driving}, 
  author       = {Di Zeng and Ling Zheng and Xiantong Yang and Yinong Li},
  year         = {2025},
  eprint       = {2410.15607},
  archivePrefix= {arXiv},
  primaryClass = {cs.RO},
  url          = {https://arxiv.org/abs/2410.15607},
  note         = {Accepted by Automotive Innovation}
}
```

