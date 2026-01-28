# Lerobot SO-1010 Firmware config

This repository contains the firmware configuration files for the Lerobot SO-1010 robot. The firmware is based on the popular open-source robotics platform and is tailored specifically for the SO-1010 model.

> Follower arm port is following:

```bash
/dev/tty.usbmodem5AB90675581
```

> To calibrate the arm, use the following command:

```bash
lerobot-calibrate \
--robot.type=so101_follower \
--robot.port /dev/tty.usbmodem5AB90675581 \
--robot.id=juns_follwer_arm
```

> Leader arm port is following:

```bash
/dev/tty.usbmodem5AB90671651
```

> To calibrate the arm, use the following command:

```bash
lerobot-calibrate \                                                                                          ─╯
    --teleop.type=so101_leader \
    --teleop.port=/dev/tty.usbmodem5AB90671651 \
    --teleop.id=juns_leader_arm
```

## Camera Configuration

| Camera | Index | Position |
|--------|:-----:|----------|
| Wrist  | 0     | Mounted on gripper |
| Top    | 1     | Overhead view of workspace |

## Getting Started

To run the teleoperation for the Lerobot SO-1010 with cameras, use the following command:

```bash
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=/dev/tty.usbmodem5AB90675581 \
    --robot.id=juns_follower_arm \
    --robot.cameras="{ top: {type: opencv, index_or_path: 1, width: 640, height: 360, fps: 30}, wrist: {type: opencv, index_or_path: 0, width: 640, height: 360, fps: 30}}" \
    --teleop.type=so101_leader \
    --teleop.port=/dev/tty.usbmodem5AB90671651 \
    --teleop.id=juns_leader_arm \
    --display_data=true
```

## Recording Datasets

To record a dataset for training, use the `lerobot-record` command. This will record episodes of you teleoperating the robot.

### Basic Recording Command

```bash
lerobot-record \
    --robot.type=so101_follower \
    --robot.port=/dev/tty.usbmodem5AB90675581 \
    --robot.id=juns_follower_arm \
    --robot.cameras="{ top: {type: opencv, index_or_path: 1, width: 640, height: 360, fps: 30}, wrist: {type: opencv, index_or_path: 0, width: 640, height: 360, fps: 30}}" \
    --teleop.type=so101_leader \
    --teleop.port=/dev/tty.usbmodem5AB90671651 \
    --teleop.id=juns_leader_arm \
    --dataset.repo_id=${HF_USER}/so101_pick_and_place \
    --dataset.single_task="Pick up the object and place it in the target location" \
    --dataset.num_episodes=50 \
    --dataset.fps=30
```

### Recording Parameters

| Parameter | Description | Recommended Value |
|-----------|-------------|-------------------|
| `--dataset.repo_id` | Hugging Face dataset repository ID | `${HF_USER}/your_dataset_name` |
| `--dataset.single_task` | Description of the task being performed | Descriptive text of your task |
| `--dataset.num_episodes` | Number of episodes to record | 50-100 for ACT |
| `--dataset.fps` | Frames per second | 30 |
| `--dataset.episode_time_s` | Max time per episode (seconds) | 60 |
| `--dataset.reset_time_s` | Time between episodes for reset | 10 |
| `--dataset.push_to_hub` | Upload to Hugging Face Hub | true |

### Tips for Recording Quality Data

1. **Consistency**: Perform the task the same way each episode
2. **Smooth motions**: Avoid jerky movements
3. **Good lighting**: Ensure consistent, adequate lighting
4. **Clear workspace**: Remove unnecessary objects
5. **Multiple demonstrations**: Record at least 50 episodes for ACT

## Training ACT

After recording your dataset, train an ACT policy using the following command:

### Basic Training Command

```bash
lerobot-train \
    --dataset.repo_id=${HF_USER}/so101_pick_and_place \
    --policy.type=act \
    --output_dir=outputs/train/act_so101_pick_and_place \
    --job_name=act_so101_pick_and_place \
    --policy.device=cuda \
    --wandb.enable=true
```

### Training on macOS (MPS)

If you're training on a Mac with Apple Silicon:

```bash
lerobot-train \
    --dataset.repo_id=${HF_USER}/so101_pick_and_place \
    --policy.type=act \
    --output_dir=outputs/train/act_so101_pick_and_place \
    --job_name=act_so101_pick_and_place \
    --policy.device=mps
```

### Training Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `--policy.type` | Policy architecture | `act` |
| `--policy.device` | Training device | `cuda`, `mps`, or `cpu` |
| `--output_dir` | Where to save checkpoints | `outputs/train/...` |
| `--wandb.enable` | Enable Weights & Biases logging | `true` |
| `--policy.n_encoder_layers` | Number of encoder layers | 4 |
| `--policy.n_decoder_layers` | Number of decoder layers | 1 |
| `--policy.dim_model` | Model dimension | 256 |

### Training Tips

- **Training time**: ~2-4 hours on a single GPU for 100k steps
- **Data efficiency**: ACT can work well with just 50 demonstrations
- **Checkpoints**: Models are saved periodically in `output_dir`

## Evaluating the Trained Policy

Once training is complete, run your trained policy on the robot:

```bash
lerobot-record \
    --robot.type=so101_follower \
    --robot.port=/dev/tty.usbmodem5AB90675581 \
    --robot.id=juns_follower_arm \
    --robot.cameras="{ top: {type: opencv, index_or_path: 1, width: 640, height: 360, fps: 30}, wrist: {type: opencv, index_or_path: 0, width: 640, height: 360, fps: 30}}" \
    --dataset.repo_id=${HF_USER}/eval_so101_pick_and_place \
    --dataset.single_task="Pick up the object and place it in the target location" \
    --dataset.num_episodes=10 \
    --policy.path=outputs/train/act_so101_pick_and_place/checkpoints/last/pretrained_model
```

### Using a Policy from Hugging Face Hub

If you've pushed your trained policy to the Hub:

```bash
lerobot-record \
    --robot.type=so101_follower \
    --robot.port=/dev/tty.usbmodem5AB90675581 \
    --robot.id=juns_follower_arm \
    --robot.cameras="{ top: {type: opencv, index_or_path: 1, width: 640, height: 360, fps: 30}, wrist: {type: opencv, index_or_path: 0, width: 640, height: 360, fps: 30}}" \
    --dataset.repo_id=${HF_USER}/eval_so101_pick_and_place \
    --dataset.single_task="Pick up the object and place it in the target location" \
    --dataset.num_episodes=10 \
    --policy.path=${HF_USER}/act_so101_pick_and_place
```

## Workflow Summary

```
1. Teleoperate    -->  Test robot movement and camera setup
2. Record         -->  Collect 50+ demonstration episodes  
3. Train          -->  Train ACT policy (~2-4 hours)
4. Evaluate       -->  Run policy on robot and measure success
```
