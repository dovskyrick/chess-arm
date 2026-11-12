# Chapter 2 — Inside LeRobot, and what robot learning really does

Each section is about 60 words. It is written against LeRobot 0.6.1, the version pinned in `env/environment.yml`. Names can change between versions.

## 2.1 The five building blocks
LeRobot is built from five pieces:
- **Robot:** the arm you control, e.g. `so101_follower`.
- **Teleoperator:** the device a human drives it with, e.g. `so101_leader`, keyboard, phone.
- **Dataset:** recorded episodes.
- **Policy:** a neural network mapping observations to actions.
- **Scripts:** command-line tools tying them together.

Almost every command is "connect a robot to a teleoperator or a policy, maybe record".

## 2.2 Robot classes
A robot class hides the hardware. The SO-101 follower (`lerobot/robots/so_follower/`) owns a Feetech motor bus and any cameras. Its key methods:
- `connect()` and `calibrate()`
- `get_observation()`: reads joint positions and camera frames
- `send_action()`: writes goal positions, capped by `max_relative_target`
- `disconnect()`

Reading this file once is the fastest way to understand how LeRobot touches hardware.

## 2.3 Observations and actions are dictionaries
An observation is a dict such as `{"shoulder_pan.pos": 12.3, ..., "gripper.pos": 40.0, "front": <image>}`. An action is a dict of the same joint keys holding *target* positions. Nothing magic: an action for the SO-101 is just six numbers, the joint goals for the next tick. Every policy, from a hand-written script to a large VLA, ultimately outputs those six numbers about 30 times per second.

## 2.4 Teleoperation
The leader arm has the same motors but no torque, so you move it by hand. Each tick, LeRobot reads the leader's joint positions and sends them as the follower's goal positions. That is the entire trick. Because both arms are calibrated to the same scale, the follower mirrors you. When recording, the leader's positions are stored as the "action" a human expert chose.

## 2.5 Configs and the command line
LeRobot uses `draccus`: Python dataclasses that double as command-line arguments. `--robot.type=so101_follower --robot.port=/dev/ttyACM0` builds an `SOFollowerRobotConfig` with those fields. The `type` names are registered with decorators in each `config_*.py`. When a command fails, open that config class. Every option you can pass lives there, with its default value.

## 2.6 The main scripts
- `lerobot-find-port`, `lerobot-setup-motors`, `lerobot-calibrate`: hardware setup.
- `lerobot-teleoperate`: drive the arm by hand.
- `lerobot-record`: teleoperate *and* save a dataset (or run a policy and save).
- `lerobot-replay`: play an episode back on the arm.
- `lerobot-train`, `lerobot-eval`: learn a policy, and test it in simulation.
- `lerobot-dataset-viz`: inspect a dataset.

## 2.7 The LeRobotDataset format
A dataset (format v3.0) is a folder:
- **Parquet files:** one row per timestep, holding state, action, timestamp, episode and frame index.
- **MP4 videos:** one per camera, which keeps datasets small.
- **`meta/`:** frames per second, feature shapes, per-episode boundaries and normalisation statistics.

`LeRobotDataset("user/name")` downloads it from the Hub. Indexing it returns PyTorch tensors, so it plugs straight into a training loop.

## 2.8 The Hugging Face Hub
The Hub is LeRobot's storage. Datasets and trained policies are git-backed repos, public or private, with the same tools and authentication (`hf auth login`). Recording can push automatically. Training can pull a dataset by name and push the resulting checkpoint. This is the "safe cloud storage" for your project: versioned, free for modest sizes, and shareable with your brother.

## 2.9 Processors and normalisation
Neural networks want inputs around zero with similar scales. LeRobot computes mean and std (or min/max) for every feature when recording. It then applies **processor pipelines**: normalise inputs, move them to the GPU, resize images, and finally un-normalise the network's output back into joint targets. Processors are saved with the policy. If you train with one calibration and run with another, actions will be silently wrong.

## 2.10 What a policy is
A policy is a function: observation in, action out. It can be a script you wrote, or a neural network trained from data. Learned policies in LeRobot usually predict an **action chunk**: the next N actions at once, not one. Chunking makes motion smoother and reduces the "hesitation" that happens when a network re-decides every single tick from slightly noisy inputs.

## 2.11 Imitation learning (behaviour cloning)
Most LeRobot policies learn by **imitation**: given the observation a human saw, predict the action the human took. It is plain supervised learning on your recorded demos. Strength: it needs no reward function and works with tens of demos. Weakness: it only knows situations similar to the demos. It copies what you did, not why, so you must record carefully.

## 2.12 Compounding errors
A cloned policy makes a small mistake. Now it sees a state no demo covered, so it makes a bigger one, and drifts away. This **distribution shift** is the core weakness of imitation. Remedies:
- More varied demos.
- Demos that include recoveries from mistakes.
- Action chunking.
- Human corrections during runs, which can be added to the data.

Expect your first policy to work sometimes and fail confusingly.

## 2.13 ACT
ACT (Action Chunking with Transformers) was introduced with the ALOHA arms. A ResNet encodes the camera images, a transformer mixes them with joint state, and it outputs a chunk (default 100 steps). During training a small variational encoder (a CVAE) captures the different styles of demos. It is fast and light (trains on one consumer GPU), and it is the usual first choice for SO-101 tasks with about 50 demos.

## 2.14 Diffusion Policy
Diffusion Policy produces actions the way image diffusion models produce pictures. It starts from noise and denoises it step by step into an action sequence, conditioned on observations. Its strength is **multimodality**: if demos sometimes go left and sometimes right, it picks one cleanly instead of averaging into a crash. Its costs are slower inference (many denoising steps), more data, and more tuning than ACT.

## 2.15 Vision-language-action models
VLAs (SmolVLA in LeRobot, about 450M parameters; the pi0 family, much bigger) start from a pretrained vision-language model. They take images plus a text instruction such as "pick the white pawn on e2" and output action chunks. Pretraining on many robots gives them better generalisation. You fine-tune on your own demos. Costs: more GPU memory and slower inference, and the language really helps only with varied data.

## 2.16 Reinforcement learning
Instead of copying, RL tries actions and learns from a **reward**. It can exceed human demos, but on a real arm it is slow, needs resets, and needs a reliable way to compute the reward. LeRobot includes HIL-SERL: RL started from demos, with a human stepping in to correct during training. Great for one narrow skill, like a precise grasp. Overkill for a chess MVP.

## 2.17 Simulation and the sim-to-real gap
Simulators (MuJoCo-based ALOHA, PushT) let you train and evaluate without hardware, reset instantly, and measure success rates honestly. But simulated physics, friction, lighting and cameras never quite match reality. A policy trained purely in simulation usually fails on the real arm. Use simulation to *learn the tools and compare methods*, not to produce your final chess policy.

## 2.18 Training and evaluation in practice
`lerobot-train` takes a dataset, a policy type and steps. It logs loss and saves checkpoints. **Low loss does not mean success.** The only real metric is the success rate on actual rollouts: in simulation via `lerobot-eval`, on hardware by running and counting. Always keep a fixed test procedure, such as "10 attempts, same start pose", so comparisons are fair.

## 2.19 Limitations that matter for chess
- **Precision:** learned policies are good at "roughly there" and weak at millimetre-exact placement on 2.5 cm squares.
- **Goal conditioning:** ACT has no built-in way to receive "go to e4". You would need 64 tasks, or language (a VLA), or an image marker.
- **No guarantees:** a learned policy can't promise safety or correctness.

This is why the MVP scripts the moves and learns later.

## 2.20 The good news
Learning shines where scripting struggles: messy grasps, pieces knocked slightly off-centre, recovering from bumps, adapting to new piece sets. A mature design often combines both. Scripted geometry gets the gripper near the square, then a small learned policy does the final grasp. Every demo you record now becomes data for that hybrid, and for your long-term video-learning idea.
