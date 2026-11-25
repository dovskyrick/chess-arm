# chess-arm

A physical chess-playing SO-101 robot arm for a small electronic-art exhibition, built on Hugging Face LeRobot. Minimal viable product above everything. Plans live in `plans/`, learning notes in `learn/`, the env in `env/`.

## Writing for the user
- **Acronym reminders:** roughly one time in three, expand robotics/ML acronyms in passing the first time they appear in a reply, e.g. "ACT (Action Chunking with Transformers)", "DM (diffusion models)", "FM (flow matching)", "VLA (vision-language-action model)", "BC (behaviour cloning)", "IK (inverse kinematics)", "HIL (human-in-the-loop)". Otherwise use the bare acronym. The user knows them, this is a light refresher.
- Guides for the user are short Markdown files with a fixed word budget per subtopic; respect the budget they ask for.

## Technical defaults
- LeRobot pinned at 0.6.1, Python 3.12, torch 2.11.0 (see `env/environment.yml`). Check commands against the pinned version, not the latest docs.
- Development machine: Windows laptop, RTX 3070 Laptop GPU (8 GB), using WSL2.
- Large data and models go to private Hugging Face Hub repos, never into git.
