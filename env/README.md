# Environment setup

**Use WSL2 (Ubuntu), not native Windows**, for this env. Reasons:
- On Linux, `pip install torch` gets a CUDA build directly from PyPI. On Windows the PyPI wheel is CPU-only and you must point pip at PyTorch's own index.
- LeRobot is developed and tested mainly on Linux; its docs and commands assume it.
- WSL2 uses your Windows NVIDIA driver. **Do not install a Linux NVIDIA driver inside WSL.**

## Steps (inside WSL)

1. On Windows: update the NVIDIA driver (version 580 or newer for CUDA 13).
2. In WSL: install Miniconda (or Miniforge) for Linux.
3. From the repo root:
   ```bash
   conda env create -f env/environment.yml
   conda activate chess-arm
   ```
4. Check:
   ```bash
   nvidia-smi
   python -c "import torch, lerobot; print(torch.__version__, torch.cuda.is_available(), lerobot.__version__)"
   lerobot-info
   ```

## If CUDA shows `False`
- Old GPU (GTX 10xx or older) or old driver: CUDA 13 does not support it. Reinstall torch from the CUDA 12.6 index:
  `pip install torch==2.11.0 torchvision==0.26.0 --index-url https://download.pytorch.org/whl/cu126`

## Hardware later (arm + webcams)
WSL does not see USB devices by default. The arm's USB-serial board can be forwarded with `usbipd-win`, which usually works. Webcams through WSL are unreliable. When you get to the real arm, either forward devices with `usbipd`, or run the hardware scripts on a native Linux machine or a Linux partition. Simulation and training stay fine in WSL.
