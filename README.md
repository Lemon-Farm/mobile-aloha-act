# Mobile ALOHA ACT

This project trains and evaluates **Action Chunking Transformer (ACT)** policies on the **Mobile ALOHA** platform using real-world teleoperated demonstrations.

The overall pipeline is:

> Teleoperation → Demonstration Collection → ACT Training → Real-World Evaluation

This repository is based on [ACT++](https://github.com/MarkFzp/act-plus-plus), with minor modifications for our Mobile ALOHA manipulation tasks.

---

## 🎬 Demo

### `cup_stack_mobile`

[![Watch Demo](./cup_stack_mobile_thumbnail.png)](./cup_stack_mobile.mp4)

Click the image above to watch the demo.

---

## 📌 Tasks

### `stack_tomato`

A dual-arm manipulation task performed with Mobile ALOHA.

- Teleoperated demonstrations: `80`
- Episode length: `750`
- Cameras:
  - `cam_high`
  - `cam_left_wrist`
  - `cam_right_wrist`

### `cup_stack_mobile`

A manipulation task that combines dual-arm control with mobile-base movement.

- Teleoperated demonstrations: `60`
- Episode length: `1250`
- Cameras:
  - `cam_high`
  - `cam_left_wrist`
  - `cam_right_wrist`

---

## 📊 Results

The trained ACT policies were evaluated directly on the **real Mobile ALOHA robot** through repeated real-world rollouts.

| Task | Success | Trials | Success Rate |
|---|---:|---:|---:|
| `stack_tomato` | `47` | `50` | `94%` |
| `cup_stack_mobile` | `43` | `50` | `86%` |

All reported results were obtained from real-world robot executions rather than simulation.

---

## 🧠 Experiment Configuration

Both tasks were trained using ACT.

| Parameter | `stack_tomato` | `cup_stack_mobile` |
|---|---:|---:|
| Training steps | `120,000` | `150,000` |
| Learning rate | `1e-5` | `2e-5` |
| Chunk size | `100` | `45` |
| KL weight | `10` | `10` |
| Hidden dimension | `512` | `512` |
| Feedforward dimension | `3200` | `3200` |
| Backbone | `ResNet-18` | `ResNet-18` |
| Seed | `0` | `0` |

Temporal aggregation was not used in either experiment.

---

## 🔧 Modification for the Mobile Task

For the `cup_stack_mobile` task, we made a small modification to:

```text
detr/models/detr_vae.py
```

The Transformer output selection was changed from:

```python
# Original
hs = self.transformer(...)[0]
```

to:

```python
# Modified
hs = self.transformer(...)[-1]
```

The same change was applied to both Transformer calls in the file.

This modification was introduced as an experimental design choice for the more complex mobile manipulation task, with the intention of using a more deeply processed Transformer representation.

> Note: We did not perform a controlled ablation study comparing `[0]` and `[-1]`, so the observed performance cannot be attributed solely to this modification.

---

## 📁 Prerequisites

To run this project on the real Mobile ALOHA robot, the following repository is also required:

- [MarkFzp/mobile-aloha](https://github.com/MarkFzp/mobile-aloha)

Place both repositories under the same parent directory:

```text
workspace/
├── mobile-aloha/
└── mobile-aloha-act/
```

Example:

```bash
git clone https://github.com/MarkFzp/mobile-aloha.git
git clone https://github.com/Lemon-Farm/mobile-aloha-act.git
```

---

## ⚙️ Task Configuration

Add the following task definitions to:

```text
mobile-aloha/aloha_scripts/constants.py
```

```python
TASK_CONFIGS = {

    'tomato': {
        'dataset_dir': DATA_DIR + '/tomato',
        'num_episodes': 80,
        'episode_len': 750,
        'camera_names': [
            'cam_high',
            'cam_left_wrist',
            'cam_right_wrist'
        ]
    },

    'cup_stack_mobile': {
        'dataset_dir': DATA_DIR + '/cup_stack_mobile',
        'num_episodes': 60,
        'episode_len': 1250,
        'camera_names': [
            'cam_high',
            'cam_left_wrist',
            'cam_right_wrist'
        ]
    },
}
```

This README assumes that the teleoperation demonstrations have already been collected.

---

## 🚀 Training

Training is performed with `imitate_episodes.py`.

### `stack_tomato`

```bash
python3 imitate_episodes.py \
    --task_name tomato \
    --ckpt_dir ./ckpt_tomato \
    --policy_class ACT \
    --batch_size <BATCH_SIZE> \
    --num_steps 120000 \
    --lr 1e-5 \
    --kl_weight 10 \
    --chunk_size 100 \
    --hidden_dim 512 \
    --dim_feedforward 3200 \
    --validate_every 3000 \
    --save_every 5000 \
    --seed 0
```

### `cup_stack_mobile`

```bash
python3 imitate_episodes.py \
    --task_name cup_stack_mobile \
    --ckpt_dir ./ckpt_cup_stack_mobile \
    --policy_class ACT \
    --batch_size <BATCH_SIZE> \
    --num_steps 150000 \
    --lr 2e-5 \
    --kl_weight 10 \
    --chunk_size 45 \
    --hidden_dim 512 \
    --dim_feedforward 3200 \
    --validate_every 3000 \
    --save_every 5000 \
    --seed 0
```

Replace `<BATCH_SIZE>` with the batch size used in the corresponding experiment.

---

## 🤖 Real-World Evaluation

The trained policies were evaluated directly on the physical Mobile ALOHA platform.

- `stack_tomato`: `47 / 50` successful trials
- `cup_stack_mobile`: `43 / 50` successful trials

The evaluation was conducted through closed-loop policy execution on the real robot.

To run evaluation, use the same configuration as training with the additional `--eval` flag.

For example:

```bash
python3 imitate_episodes.py \
    --eval \
    --task_name cup_stack_mobile \
    --ckpt_dir ./ckpt_cup_stack_mobile \
    --policy_class ACT \
    --batch_size <BATCH_SIZE> \
    --num_steps 150000 \
    --lr 2e-5 \
    --kl_weight 10 \
    --chunk_size 45 \
    --hidden_dim 512 \
    --dim_feedforward 3200 \
    --seed 0
```

Because evaluation directly controls the physical robot, verify the robot configuration, workspace, and emergency stop before execution.

---

## 🔗 References

This project builds upon the following repositories:

- [ACT++](https://github.com/MarkFzp/act-plus-plus)
- [Mobile ALOHA](https://github.com/MarkFzp/mobile-aloha)
- [ACT](https://github.com/tonyzhaozh/act)

---

## 📄 License

This repository follows the MIT License of the original [ACT++](https://github.com/MarkFzp/act-plus-plus) implementation.

See [`LICENSE`](./LICENSE) for details.
