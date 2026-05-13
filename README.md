# 🦾 SO-ARM101 / SO101 — Robot Control & AI Policy Guide

A complete guide for setting up, calibrating, teleoperation, dataset recording, and AI policy training for the **SO101 robotic arm** using the [LeRobot](https://github.com/huggingface/lerobot) framework.

---

## 📋 System Overview

| Parameter | Value |
|---|---|
| Conda Environment | `lerobot` |
| Follower Robot | `so101_follower` — ID: `pippo` — Port: `COM3` |
| Leader Robot | `so101_leader` — ID: `paperino` — Port: `COM4` |
| Cameras (OpenCV) | `front` → index `1`, `front1` → index `2` |
| Resolution / FPS | 640×480 @ 30 fps |
| Hugging Face User | `SOARM1` |

---

## ⚡ Quick Start — Recommended Sequence

```bash
# 1. Activate the Conda environment
conda activate lerobot

# 2. Find the serial port
lerobot-find-port

# 3. Setup motors (follower)
lerobot-setup-motors --robot.type=so101_follower --robot.port=COM3

# 4. Calibrate the follower
lerobot-calibrate --robot.type=so101_follower --robot.port=COM3 --robot.id=pippo

# 5. Calibrate the leader
lerobot-calibrate --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=paperino

# 6. Find cameras
lerobot-find-cameras opencv

# 7. Start teleoperation
lerobot-teleoperate \
  --robot.type=so101_follower --robot.port=COM3 --robot.id=pippo \
  --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=paperino
```

---

## 🔧 Setup Details

### 1. Activate Conda Environment

```bash
conda activate lerobot
```

### 2. Find Serial Port

```bash
lerobot-find-port
```

### 3. Motor Setup

```bash
# Follower on COM3
lerobot-setup-motors --robot.type=so101_follower --robot.port=COM3

# Alternative port
lerobot-setup-motors --robot.type=so101_follower --robot.port=COM5
```

### 4. Motor Calibration

**Follower (`pippo`):**
```bash
lerobot-calibrate --robot.type=so101_follower --robot.port=COM3 --robot.id=pippo
```

**Leader (`paperino`):**
```bash
lerobot-calibrate --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=paperino
```

---

## 📷 Cameras & Teleoperation

### Find Cameras

```bash
lerobot-find-cameras opencv
```

### Teleoperation with Dual Cameras

```bash
lerobot-teleoperate \
  --robot.type=so101_follower --robot.port=COM3 --robot.id=pippo \
  --robot.cameras="{ front: {type: opencv, index_or_path: 1, width: 640, height: 480, fps: 30}, \
                     front1: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30}}" \
  --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=paperino \
  --display_data=true
```

---

## 📦 Dataset Recording

### Hugging Face Authentication

```bash
# Login with token
hf auth login --token <HF_TOKEN> --add-to-git-credential

# Set user variable (Windows)
set HF_USER=SOARM1
echo %HF_USER%

# Set user variable (Linux/macOS)
HF_USER=$(hf auth whoami | awk -F': *' 'NR==1 {print $2}')
echo $HF_USER
```

### Record a Dataset

```bash
lerobot-record \
  --robot.type=so101_follower --robot.port=COM3 --robot.id=pippo \
  --robot.cameras="{ front: {type: opencv, index_or_path: 1, width: 640, height: 480, fps: 30}, \
                     front1: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30}}" \
  --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=paperino \
  --display_data=true \
  --dataset.repo_id=${HF_USER}/record-test \
  --dataset.num_episodes=5 \
  --dataset.single_task="Grab the black cube"
```

### Advanced Recording (custom task)

```bash
lerobot-record \
  --robot.type=so101_follower --robot.port=COM3 --robot.id=pippo \
  --robot.cameras="{ front: {type: opencv, index_or_path: 1, width: 640, height: 480, fps: 30}, \
                     front1: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30}}" \
  --teleop.type=so101_leader --teleop.port=COM4 --teleop.id=paperino \
  --display_data=true \
  --dataset.repo_id=SOARM1/record-test1 \
  --dataset.num_episodes=10 \
  --dataset.single_task="cubo_arancione"
```

### Local Save Paths (Windows)

```
C:\Users\donno\outputs\captured_images
C:\Users\donno\.cache\huggingface\lerobot\<HF_USER>
```

### View Dataset on Hugging Face

```bash
echo https://huggingface.co/datasets/${HF_USER}/so101_test
# https://huggingface.co/datasets/SOARM1/so101_test
```

---

## ☁️ Upload Dataset to Hugging Face

Install the Hugging Face CLI (Windows PowerShell):

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://hf.co/cli/install.ps1 | iex"
```

Login and upload:

```bash
hf auth login

hf upload SOARM1/cubo_arancione . --repo-type=dataset
```

---

## 🤖 AI Model Training (Google Colab)

Train an **ACT policy** on the merged dataset:

```python
!cd lerobot && python src/lerobot/scripts/lerobot_train.py \
  --dataset.repo_id=SOARM1/SOARM101_merged \
  --policy.type=act \
  --output_dir=outputs/train/hf_act_record0 \
  --job_name=hf_act_training_job \
  --policy.device=cuda \
  --wandb.enable=False \
  --policy.repo_id=SOARM1/hf_act_recordpolicy0
```

**Key training parameters:**

| Parameter | Value |
|---|---|
| Dataset | `SOARM1/SOARM101_merged` |
| Policy type | `ACT` |
| Device | `cuda` |
| Output dir | `outputs/train/hf_act_record0` |
| Policy repo | `SOARM1/hf_act_recordpolicy0` |
| W&B logging | Disabled |

### Advanced Training (custom chunk & action steps)

```bash
lerobot-train \
  --dataset.repo_id=giovipeg/fiera_merged \
  --policy.type=act \
  --output_dir=outputs/train/fiera50steps01 \
  --job_name=fiera50steps01 \
  --policy.device=cuda \
  --wandb.enable=true \
  --policy.repo_id=giovipeg/fiera50steps01 \
  --policy.chunk_size=50 \
  --policy.n_action_steps=10
```

> `chunk_size=50` — number of steps managed in sequence  
> `n_action_steps=10` — number of actions predicted by the policy  
> These settings improve temporal continuity and produce smoother real-world robot motion.

---

## 🎮 Running the Policy on the Robot

Load a trained policy directly into `lerobot-record` for autonomous execution:

```bash
lerobot-record \
  --robot.type=so100_follower --robot.port=COM3 --robot.id=pippo \
  --robot.cameras="{ front: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30}, \
                     front1: {type: opencv, index_or_path: 1, width: 640, height: 480, fps: 30}}" \
  --display_data=true \
  --dataset.repo_id=SOARM1/eval_b \
  --dataset.single_task="Put lego brick into the box" \
  --policy.path=giovipeg/fiera50stepsNoFront
```

---

## ✂️ Dataset Editing

Remove an unwanted feature (e.g. a camera) from an existing dataset:

```bash
lerobot-edit-dataset \
  --repo_id=giovipeg/fiera_merged \
  --new_repo_id=giovipeg/fiera_merged_no_side \
  --operation.type=remove_feature \
  --operation.feature_names="['observation.images.front1']" \
  --push_to_hub=true
```

---

## 🔍 Dataset Visualization

Inspect a specific episode from a recorded dataset:

```bash
lerobot-dataset-viz --repo-id SOARM1/SOARM101_merged --episode-index 0
```

Useful for verifying:
- Image quality
- Correctness of movements
- Observation synchronization
- Recording errors

---

## 🔗 Resources

- 🤗 Hugging Face profile: [SOARM1](https://huggingface.co/SOARM1)
- 📦 Dataset example: [SOARM1/so101_test](https://huggingface.co/datasets/SOARM1/so101_test)
- 🤖 LeRobot framework: [github.com/huggingface/lerobot](https://github.com/huggingface/lerobot)
