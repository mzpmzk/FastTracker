# FastTracker

#### FastTracker is a real-Time and accurate visual tracking module.

<div align="center">

[**Hamidreza Hashempoor**](https://hamidreza-hashempoor.github.io/)


<!-- **TMLCN 2025** -->

</div>



<div align="center">
  <img src="./figs/tracker_radar.jpg" alt="Image main" width="30%" style="margin: 1%;">
</div>

FastTracker is a general-purpose multi-object tracking framework designed for complex traffic scenes. FastTracker supports diverse object types—especially vehicles—and maintains identity through heavy occlusion and complex motion. It combines an occlusion-aware re-identification module with road-structure-aware tracklet refinement, using semantic priors like lanes and crosswalks for better trajectory accuracy.
## Resources
| Huggingface Dataset | Paper |
|:-----------------:|:-------:|
|[![Hugging Face Spaces](https://img.shields.io/badge/HuggingFace-Dataset-blue)](https://huggingface.co/datasets/Hamidreza-Hashemp/FastTracker-Benchmark)|[arXiv ](xxx)




## Benchmark
FastTrack is a high-density multi-object tracking benchmark tailored for complex urban traffic scenes. It features 800K annotations across 12 diverse scenarios with 9 object classes, offering over 5× higher object density than existing benchmarks—making it ideal for evaluating trackers under extreme occlusion, interaction, and scene variety.
The Benchmark is public and available in our [**Huggingface Dataset**](https://huggingface.co/datasets/Hamidreza-Hashemp/FastTracker-Benchmark)

![image](./figs/fasttrack_benchmark.jpg)


## Framework

Occlusion-aware tracking strategy framework that detects occluded tracklets based on center-proximity with nearby objects. Once detected, occluded tracklets are marked inactive, their motion is dampened to prevent drift, and their bounding boxes are slightly enlarged to aid re-identification. 


<img src="./figs/fasttrack_occ_alg.jpg" alt="Occlusion Algorithm" style="width:70%;"/>

## Tracking performance
### Results on MOT challenge test set
| Dataset    | MOTA | IDF1 | HOTA | FP    | FN     | IDs |
|------------|------|------|------|-------|--------|-----|
| MOT16      | 79.1 | 81.0 | 66.0 | 8785  | 29028  | 290 |
| MOT17      | 81.8 | 82.0 | 66.4 | 26850 | 75162  | 885 |
| MOT20      | 77.9 | 81.0 | 65.7 | 24590 | 89243  | 684 |
| FastTracker| 63.8 | 79.2 | 61.0 | 29730 | 68541  | 251 |

## Installation on the host machine

Steps: Setup the environment
```shell
cd <home>
conda create --name FastTracker python=3.9
conda activate FastTracker
pip3 install -r requirements.txt  # Ignore the errors
python setup.py develop
pip3 install cython
conda install -c conda-forge pycocotools
pip3 install cython_bbox
```



## Data preparation

Download [MOT16](https://motchallenge.net/), [MOT17](https://motchallenge.net/), [MOT20](https://motchallenge.net/), [FastTracker](https://huggingface.co/datasets/Hamidreza-Hashemp/FastTracker-Benchmark) and put them under `./datasets` in the following structure:
```
datasets
   |——————FastTracker
   |        └——————train
   |        └——————test
   |——————MOT16
   |        └——————train
   |        └——————test
   |——————mot
   |        └——————train
   |        └——————test
   └——————MOT20
            └——————train
            └——————test

```

Then, you need to turn the datasets to COCO format and mix different training data:

```shell
cd <home>
python tools\\convert_mot16_to_coco.py
python tools\\convert_mot17_to_coco.py 
python tools\\convert_mot20_to_coco.py
```
(For FastTracker benchmark use `convert_mot17_to_coco.py` to make annotations. There you need to change
`DATA_PATH = 'datasets/mot'` line.)

## Pretrained Weights

To use FastTracker's default models for MOT17 and MOT20 benchmarks, download the following pretrained weights from the [ByteTrack repository](https://github.com/ifzhang/ByteTrack):

- `bytetrack_x_mot17.pth.tar`  
- `bytetrack_x_mot20.pth.tar`  

Place both files into the `./pretrained/` directory.

For the **FastTrack benchmark**, which includes multiple object classes beyond pedestrians, you need to retrain the YOLOX backbone for multi-class detection. 

The FastTrack benchmark uses the standard MOT format, where each `gt/gt.txt` file already provides frame-level detections with object class annotations. To train a detector:

1. You **do not need to extract frames manually**—the frame-wise detections are already specified.
2. Simply **ignore the track IDs** in `gt.txt`, as detector training requires only bounding boxes and class labels.
3. Convert the annotations to COCO-style format using the provided class and bounding box info.
4. Use the ByteTrack training scripts to retrain YOLOX with the new annotations.

This enables FastTracker to detect and track multiple object types effectively in complex traffic environments.


## Tracking

Run FastTracker:

To evaluate **FastTracker** on the **MOT17 benchmark** and **MOT20 benchmark**, simply run the following command:

```bash
bash run_mot17.sh
bash run_mot20.sh
```
For  **MOT16 benchmark** and **FatTracker benchmark**, you can use `bash run_mot17.sh`, but need to change the weight directory, experiment name and experiment file.

### ⚙️ Understanding `run_mot17.sh` and `run_mot20.sh` Configuration:

The `run_mot17.sh` script begins by defining a set of key variables that control the evaluation pipeline:

```bash
EXP_FILE="exps/example/xx"
CKPT="pretrained/bytetrack_x_motxx.pth.tar"
BATCH=1
DEVICES=1
EXPERIMENT_NAME="xx"
OUTPUTS_ROOT="YOLOX_outputs"
CONFIGS_DIR="./configs"
```

Here is the defenition of each variable:

* `EXP_FILE`:
Path to the YOLOX experiment definition file. This file specifies the model architecture, dataset format, and preprocessing.
Example: `exps/example/mot/yolox_x_mix_det.py`.

* `CKPT`:
Path to the pretrained checkpoint file. For `MOT17` or `MOT20`, you must download the appropriate `.pth.tar` file from the ByteTrack repository
 and place it under the `./pretrained/` directory.
Examples: `pretrained/bytetrack_x_mot17.pth.tar` or `pretrained/bytetrack_x_mot20.pth.tar`

* `BATCH`:
The batch size used during inference. Typically set to `1` for tracking tasks.

* `DEVICES`:
Number of GPUs to use. Set to `1` for single-GPU setups. Can be increased for parallel evaluation.

* `EXPERIMENT_NAME`:
A unique name identifying the current experiment.
The outputs (logs, tracking results, etc.) will be saved under `YOLOX_outputs/<EXPERIMENT_NAME>/`.

* `OUTPUTS_ROOT`:
The root directory where experiment outputs are saved.

* `CONFIGS_DIR`:
Directory containing multiple `.json` configuration files. Each config file defines tracking hyperparameters such as thresholds, buffer sizes, and occlusion handling settings.
The script automatically runs tracking once per config file inside this folder.

### 📄 Details of Tracking Configuration Files (./configs/*.json)
Each JSON file inside the `./configs` directory specifies a different set of hyperparameters for tracking evaluation.

* `track_thresh`:
Minimum detection score for initializing or updating a track. Lowering this allows weaker detections to be considered.

* `track_buffer`:
Maximum number of frames a tracklet is kept alive without receiving a matching detection.

* `match_thresh`:
IOU threshold used for associating detections to existing tracklets.

* `min_box_area`:
Minimum area of bounding boxes to be considered for tracking. Useful for filtering out tiny detections.

* `reset_velocity_offset_occ`:
Velocity smoothing offset applied when a tracklet is marked as occluded. Higher values help reduce drift.

* `reset_pos_offset_occ`:
Position smoothing offset for occluded objects. Controls how much the predicted position can shift.

* `enlarge_bbox_occ`:
Factor to enlarge bounding boxes of occluded objects. Helps improve re-identification during reappearance.

* `dampen_motion_occ`:
Dampening factor for the velocity vector of occluded objects. Reduces aggressive forward prediction.

* `active_occ_to_lost_thresh`:
Number of frames an occluded object can remain unmatched before being marked as lost.

* `init_iou_suppress`:
IOU suppression threshold used to avoid initializing duplicate tracks from overlapping detections.


## Obtain MOTA /IDS/ HOTA and other evaluation

### MOT17
```shell
cd <home>
python .\\TrackEval\\hotaPreparation.py -d .\\YOLOX_outputs\\yolox_x_mix_det\\track_results\\MOT17-02-DPM.txt -g .\\YOLOX_outputs\\gt\\MOT17\\02-DPM\\gt.txt
python .\\TrackEval\\scripts\\run_mot_challenge.py --USE_PARALLEL False --METRICS CLEAR --BENCHMARK MOT15
```

### MOT20
```shell
cd <home>
python .\\TrackEval\\hotaPreparation.py -d .\\YOLOX_outputs\\yolox_x_mix_mot20_ch\\track_results\\MOT20-01.txt -g .\\YOLOX_outputs\\gt\\MOT20\\01\\gt.txt
python .\\TrackEval\\scripts\\run_mot_challenge.py --USE_PARALLEL False --METRICS CLEAR --BENCHMARK MOT15
```
