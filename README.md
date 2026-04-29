<div align="center">

# CDP-Train

<p>
  <a href="https://github.com/Sihang-Geng/CDP-Train/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/badge/license-AGPL--3.0-blue"></a>
  <img alt="Python" src="https://img.shields.io/badge/python-3.8%2B-3776AB?logo=python&logoColor=white">
  <img alt="Base" src="https://img.shields.io/badge/base-Ultralytics%20YOLO-111111">
  <img alt="Metric" src="https://img.shields.io/badge/metric-COCO%20AP%20aligned-2E7D32">
  <img alt="Release" src="https://img.shields.io/badge/release-research%20code-orange">
</p>

</div>

> **Notice**  
> Research code built on [Ultralytics](https://github.com/ultralytics/ultralytics). Upstream copyright notices and the GNU AGPL-3.0 license are retained.

## 🎯 COCO-Aligned Evaluation Protocol

> **Goal:** reduce unfair comparison caused by fixed-epoch training in detection experiments.

| Icon | Common issue | Impact |
| --- | --- | --- |
| 📏 | `mAP` != COCO AP | A higher internal `mAP` may not mean a higher reported AP. |
| ⏱️ | AP peaks at different epochs | Early-converging models may be evaluated after overfitting. |
| ⚖️ | Fixed final checkpoint | Results mix model quality with convergence timing. |

**CDP-Train** runs COCO evaluation during training and keeps the best-AP checkpoint.

| Capability | Function |
| --- | --- |
| COCO AP selection | Uses COCO `mAP50-95(B)` as fitness. |
| Periodic COCO eval | Runs COCO API every `N` epochs. |
| Best checkpoint guard | Blocks non-COCO epochs from replacing `best.pt`. |
| COCO image-id mapping | Aligns prediction `image_id` with annotation JSON. |

In short, training-time checkpoint selection follows the evaluation metric used in detection papers.

## 🛠️ Implementation

Detailed changes are in [`FAIR_COMPARISON_IMPLEMENTATION.md`](FAIR_COMPARISON_IMPLEMENTATION.md). In brief, this release adds:

- COCO API evaluation during training.
- COCO-based `best.pt` selection.
- Custom COCO JSON lookup.
- Annotation-based image-id mapping.
- Optimizer fallback for easier environment setup.
- Utility scripts for qualitative visualization and figure-style analysis.

## 🚀 Installation

### 1. Clone the repository

```bash
git clone https://github.com/Sihang-Geng/CDP-Train.git
cd CDP-Train
```

### 2. Create the environment

```bash
conda create -n CDP python=3.10 -y
conda activate CDP
```

### 3. Install dependencies

Install the PyTorch version that matches your CUDA environment first if it is not already available.

```bash
# Example for CUDA 12.1
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

pip install -e .
pip install pycocotools
```

For other CUDA versions, use the selector on the [official PyTorch installation page](https://pytorch.org/get-started/locally/).

## 🗂️ Dataset

This project uses the RUOD dataset: [RUOD on Baidu AI Studio](https://aistudio.baidu.com/datasetdetail/216919).

Keep your RUOD files in a COCO-style structure and point your YAML to local paths:

```text
RUOD/
├─ images/
│  ├─ train/
│  └─ val/
└─ annotations/
   ├─ instances_train.json
   └─ instances_val.json
```

## ⚡ Training

Run the example entry:

```bash
python ultralytics/train.py
```

Minimal training example:

```python
from ultralytics import YOLO

model = YOLO("/root/ultralytics/ultralytics/cfg/models/v8/yolov8s.yaml")

results = model.train(
    data="/root/ultralytics/ultralytics/cfg/datasets/RUOD/RUOD_YOLO/data.yaml",
    epochs=250,
    imgsz=640,
    seed=0,
    deterministic=True,
    save_json=True,
    use_coco_fitness=True,
    coco_eval_interval=5,
    coco_only_best=True,
    coco_start_epoch=100,
    patience=100,
)

results = model.val()
```

## 🗂️ COCO JSON Compatibility

The validator searches common annotation locations:

```text
{data_path}/instances_val2017.json
{data_path}/annotations/instances_val2017.json
{data_path}/annotations/instances_val.json
{data_path}/annotations/instances_{split}.json
{data_path}/val/_annotations.coco.json
{data_path}/instances_val.json
{data_path}/_annotations.coco.json
```

For custom filenames, an annotation-based image ID map is built:

```python
self.img_id_map[Path(img["file_name"]).name] = img["id"]
self.img_id_map[Path(img["file_name"]).stem] = img["id"]
```

This avoids AP mismatch when filenames are not numeric COCO IDs.

## 🧪 Recommended Modes

| Mode | Use case | Key settings |
| --- | --- | --- |
| CDP-style fair comparison | Paper experiments and ablation studies. | `save_json=True`, `use_coco_fitness=True`, `coco_eval_interval=5`, `coco_only_best=True` |
| Fast pipeline check | Debug whether training runs. | `save_json=False`, `use_coco_fitness=False` |

## 🧭 Feature Map

| Module | File | Role |
| --- | --- | --- |
| Trainer | [`ultralytics/engine/trainer.py`](ultralytics/engine/trainer.py) | COCO fitness and `best.pt` control. |
| Validator | [`ultralytics/engine/validator.py`](ultralytics/engine/validator.py) | Scheduled JSON and COCO API calls. |
| Training | [`ultralytics/train.py`](ultralytics/train.py) | Example training entry. |
| Visualization | [`ultralytics/visual.py`](ultralytics/visual.py) | Qualitative detection view. |
| Plotting / 3D | [`ultralytics/plotfig2.py`](ultralytics/plotfig2.py), [`ultralytics/3d.py`](ultralytics/3d.py) | Figure plotting and 3D visualization. |
| Notes | [`FAIR_COMPARISON_IMPLEMENTATION.md`](FAIR_COMPARISON_IMPLEMENTATION.md) | Full implementation notes. |

## 📄 License

This project is released under the GNU AGPL-3.0 license inherited from Ultralytics. See [LICENSE](LICENSE).

Upstream project: [ultralytics/ultralytics](https://github.com/ultralytics/ultralytics)
