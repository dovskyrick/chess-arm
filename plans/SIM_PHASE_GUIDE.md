# While the arms ship — a simulation-first guide

About 40 words per topic. All commands are for LeRobot 0.6.1 inside WSL with the `chess-arm` conda env active. Do them in order, and note anything surprising in `docs/lerobot-notes.md`.

> **LeRobot 0.6.1 bug:** parallel simulation workers fail with `gym id ... not found`. Always pass `--eval.use_async_envs=false` (already in the commands below).

## 1. Goal of these weeks
Become fluent with LeRobot's moving parts before hardware arrives: datasets, policies, training, evaluation, kinematics. Results in simulation don't matter. Understanding what each command does, which files it writes, and where the code lives matters a lot.

## 2. Build the environment
Create the env, then confirm the GPU is visible. If CUDA prints `False`, fix that first using `env/README.md`. Everything later depends on a working GPU, and debugging installs mid-experiment is miserable. The first install downloads several gigabytes; let it finish.
```bash
conda env create -f env/environment.yml && conda activate chess-arm
python -c "import torch, lerobot; print(torch.cuda.is_available(), lerobot.__version__)"
```

## 3. Log in to the Hub
Create a Hugging Face account and an access token with write permission, then log in once. Public datasets and pretrained policies download from the Hub automatically, and your own checkpoints and recordings will be pushed there later as private repos.
```bash
hf auth login
```

## 4. Look at a dataset
Open a PushT episode in the Rerun viewer. Watch the image, the agent position and the action side by side. Notice how action leads state slightly: the action is the target, the state is where it got.
```bash
lerobot-dataset-viz --repo-id lerobot/pusht --episode-index 0
```

## 5. Load it in Python
In a notebook, load the same dataset and inspect one frame: keys, tensor shapes, fps, episode boundaries, stored statistics. This is the same object training uses, so understanding it explains half of LeRobot.
```python
from lerobot.datasets import LeRobotDataset
ds = LeRobotDataset("lerobot/pusht"); print(ds); print(ds[0].keys(), ds.meta.stats.keys())
```

## 6. Run a pretrained DM policy
Evaluate the pretrained Diffusion Policy on PushT. It writes success rate and rollout videos under `outputs/eval/`. Watch the videos closely: smooth, confident pushing is what a well-trained policy looks like. Note how long each episode takes to compute on your GPU.
```bash
lerobot-eval --policy.path=lerobot/diffusion_pusht --env.type=pusht \
  --eval.n_episodes=10 --eval.batch_size=10 --eval.use_async_envs=false --policy.device=cuda
```

## 7. Run a pretrained ACT policy
Now an arm: ACT on ALOHA's bimanual cube transfer. This one renders MuJoCo physics, which is heavier, so use small batches. Compare its motion style with diffusion: chunked, decisive, sometimes slightly jerky between chunks. If rendering fails in WSL, prefix the command with `MUJOCO_GL=egl`.
```bash
lerobot-eval --policy.path=lerobot/act_aloha_sim_transfer_cube_human --env.type=aloha \
  --env.task=AlohaTransferCube-v0 --eval.n_episodes=5 --eval.batch_size=1 --policy.device=cuda
```

## 8. Train your own ACT
Train ACT on the same human demos for a short run, saved locally, not pushed. Watch the loss curve and checkpoint folders, then evaluate a checkpoint like topic 7. Expect weaker results than pretrained; that's fine.
```bash
lerobot-train --dataset.repo_id=lerobot/aloha_sim_transfer_cube_human --policy.type=act \
  --env.type=aloha --env.task=AlohaTransferCube-v0 --steps=20000 --batch_size=8 \
  --policy.push_to_hub=false --eval.use_async_envs=false --output_dir=outputs/train/act_cube
```

## 9. Run small experiments
Change one thing per run: fewer demos, a smaller `chunk_size`, no VAE, fewer steps. Record success rates in a table. This builds intuition for how data and chunking affect behaviour, which you'll need with real demos later.

## 10. Read the code paths
Trace three paths in the source:
- `lerobot-eval`: env step → `select_action` → action queue.
- ACT's `modeling_act.py` forward pass.
- `so_follower.py`: `get_observation` and `send_action`.

Draw each as a small diagram in your notes.

## 11. The SO-101 in simulation
Download the SO-101 URDF from TheRobotStudio's SO-ARM100 repo. Load it with LeRobot's `RobotKinematics` (placo). Compute FK for a few poses, then IK for points on a virtual 20 cm board. Check that every corner is reachable with the gripper pointing down.

## 12. Start the real repo code
Begin `src/chess_arm/`:
- `engine/`: python-chess wrapper, move → pick/place steps.
- `arm/`: the `Arm` interface, with a `SimArm` that uses your IK.
- `perception/`: a board-square grid from photos of any real chessboard.

Add tests from day one.

## 13. Prepare for arrival
Before the box comes, have ready:
- the commands for finding the port, setting up the motors, calibrating and teleoperating, written in your notes;
- `usbipd-win` installed, for USB in WSL;
- a printed 20 cm board and two cameras chosen;
- a clear, stable table.

The first hardware day should be execution.

## 14. Rhythm
Aim for a few focused sessions a week. End each with one line in your notes: what worked, what broke, what's next. When stuck for more than an hour, search LeRobot's GitHub issues and Discord before digging deeper alone.
