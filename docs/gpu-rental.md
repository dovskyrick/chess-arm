# Renting GPUs for training (as of Nov 2026)

Your RTX 3070 (8 GB) handles ACT and Diffusion. Rent only for SmolVLA, pi0-class models, or to speed up long runs. Prices change constantly: compare per-hour rates on the day you rent.

1. **Hugging Face Jobs (first choice).** LeRobot 0.6.1 submits training there directly: add `--job.target=<flavor>` to `lerobot-train`, and list flavors and prices with `hf jobs hardware`. Data and checkpoints already live on the Hub, so nothing gets copied around. Caveat: the default container image is `lerobot-gpu:latest`, which may be newer than your pinned version.
2. **RunPod.** Cheap per-second GPU pods (RTX 4090, L40S, A100, H100) with a persistent volume and a Jupyter or SSH terminal. Best for hands-on experiments with full control.
3. **Vast.ai.** A marketplace of individual hosts, often the cheapest. Reliability varies, so save checkpoints to the Hub often.
4. **Lambda / Modal.** Lambda: simple cloud machines with a fixed hourly price. Modal: Python-defined serverless GPU jobs, pay per second, good once training is scripted.
5. **Kaggle / Google Colab.** Free or cheap tiers, fine for tutorials, but session limits make them bad for real runs.

**Rules:** never leave a pod running idle, set a timeout, push checkpoints to a private Hub repo, and keep API keys out of git.
