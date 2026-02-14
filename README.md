# Animal Detector

A YOLOv8-based animal detection system with ensemble classification that extracts frames from videos, trains custom models, and tracks animal movements. This project provides a complete workflow from video processing to motion tracking using YOLO for detection and AlexNet/Inception for classification refinement.

## Overview

This project uses YOLOv8 (You Only Look Once) for object detection, combined with fine-tuned AlexNet and Inception v3 classifiers for ensemble-based animal detection and tracking in video footage. The notebook-based workflow guides you through:
- Extracting frames from video files
- Preparing datasets for training (detection and classification)
- Training a custom YOLOv8 detector
- Fine-tuning AlexNet and Inception classifiers on cropped regions
- Running ensemble inference on new images
- Tracking animal positions and measuring distances traveled
- Reviewing results in a Power BI report
- Inspecting an example run of center-coordinate outputs with per-model confidences

## Recent Changes

- **Ensemble Implementation**: Added AlexNet and Inception v3 classifiers alongside YOLO for improved detection accuracy. Ensemble voting filters detections based on configurable thresholds.
- **Classification Dataset Builder**: New step to crop bounding-box regions from YOLO labels and generate positive/negative samples for classifier training.
- **Per-Model Confidence Outputs**: Inference results now include individual confidence scores from YOLO, AlexNet, and Inception for detailed analysis.
- **Added `detector.ipynb`**: Dedicated inference notebook for videos, individual frames, or mixed folders with configurable output FPS and annotated video generation.
- **Implemented video input configuration**: Supports input types (`'video'`, `'frames'`, `'folder'`) and produces labeled outputs with class names and confidence scores.
- **Improved inference robustness**: Added stronger input/model validation, deterministic output run naming, safer video/image output handling, and consistent CSV summary generation.
- **Hardened `detector_trainer.ipynb` logic**: Added path and parameter validation, safer model initialization, train/val split guardrails, and resilient fallback behavior for exports and optional views.
- **Updated documentation**: Clarified workflow boundaries and usage expectations across notebooks.

**Author**: GitHub Copilot  
**Review Status**: Human reviewed

## Which Notebook to Use

- Use `detector_trainer.ipynb` when you need to prepare data, train/validate models, and generate training outputs.
- Use `detector.ipynb` when you already have weights and want inference-only workflows (video, frames, mixed folders, labeled output video, and CSV summaries).
- Keep both notebooks if you do both model development and production-style inference; keep only `detector.ipynb` if you only run detection.

## Quick Start Matrix

| Task | Notebook | Where to Start |
|---|---|---|
| Prepare dataset folders and split labels/images | `detector_trainer.ipynb` | Step 1 → Step 3 |
| Extract evenly spaced frames from raw videos | `detector_trainer.ipynb` | Step 2 |
| Build classification dataset from YOLO labels | `detector_trainer.ipynb` | Step 3b |
| Train YOLO detector | `detector_trainer.ipynb` | Step 4a |
| Fine-tune AlexNet classifier | `detector_trainer.ipynb` | Step 4c |
| Fine-tune Inception classifier | `detector_trainer.ipynb` | Step 4d |
| Configure ensemble settings | `detector_trainer.ipynb` | Step 4e |
| Validate models and inspect training metrics | `detector_trainer.ipynb` | Step 4a–4d |
| Run ensemble inference on a single image/folder | `detector_trainer.ipynb` | Step 5 |
| Measure movement distance across ordered frames (ensemble) | `detector_trainer.ipynb` | Step 6 |
| Run inference on video, frame folder, or mixed folder | `detector.ipynb` | Cells 2–9 (config → processing) |
| Generate labeled output video at target FPS | `detector.ipynb` | Processing cell + FPS helper functions |
| Export per-frame center/confidence/distance CSV | `detector.ipynb` | Final summary/export cell |

## Prerequisites

- Python 3.7 or higher
- Jupyter Notebook or JupyterLab

## Installation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/lmansf/Animal-Detector.git
   cd Animal-Detector
   ```

2. **Install dependencies**:
   ```bash
   pip install ultralytics opencv-python torch torchvision jupyter matplotlib
   ```

3. **Launch Jupyter Notebook**:
   ```bash
   jupyter notebook detector_trainer.ipynb
   ```

## Usage

### Step 1: Setup and Paths

Configure your project paths in the notebook:
- Set `video_folder` to the location of your video files
- Set `frames_per_video` to control sampling rate
- Set `dataset_root` for output location

### Step 2: Extract Frames from Videos

1. Place your video files in the configured `video_folder`
2. Run the frame extraction cell
3. Frames will be saved in per-video subfolders

### Step 3: Annotate Your Data

Use a YOLO-compatible labeling tool to annotate the extracted frames:
- [MakeSenseAi](https://www.makesense.ai/) (this is what I used, without the ai tooling)
- [LabelImg](https://github.com/heartexlabs/labelImg)
- [Roboflow](https://roboflow.com/)

Annotations should be in YOLO format (`.txt` files with class and bounding box coordinates).

### Step 3b: Build Classification Dataset

1. After preparing the YOLO dataset, run this step to crop bounding-box regions as positive samples
2. Random background patches are generated as negative samples
3. Images are resized for AlexNet/Inception compatibility

### Step 4a: Train YOLO Detector

1. Choose a base model (default: `yolov8s.pt` - small model for balanced speed/accuracy)
2. Configure training parameters (epochs, image size, etc.)
3. Run the training cell
4. Monitor training progress and metrics
5. Best model weights are saved in `runs/detect/train*/weights/best.pt`

### Step 4b: Shared Classifier Training Utilities

Reusable dataset loaders, transforms, and training loop for AlexNet and Inception.

### Step 4c: Fine-tune AlexNet

1. Pre-trained AlexNet is loaded and its final layer replaced for binary classification
2. Trained on cropped positive/negative samples from Step 3b
3. Best weights saved in `runs/classify/alexnet_best.pt`

### Step 4d: Fine-tune Inception v3

1. Pre-trained Inception v3 is loaded and adapted for binary classification
2. Uses 299×299 input size, trained on the same dataset as AlexNet
3. Best weights saved in `runs/classify/inception_best.pt`

### Step 4e: Configure Ensemble

1. Set the ensemble threshold (default: 0.5)
2. Ensemble combines YOLO detection with classifier voting

### Step 5: Run Ensemble Inference

Test your trained ensemble:
1. Select an image or folder for testing
2. Load trained YOLO, AlexNet, and Inception weights
3. Run detection to see bounding boxes and per-model confidence scores
4. Only detections passing the ensemble threshold are reported

### Step 6: Measure Distance Traveled (Ensemble)

Track animal movement across video frames using ensemble detection:
1. Provide a folder of ordered frames (from a video)
2. Run ensemble detection on all frames
3. Calculate distances between detected positions
4. Results include per-model confidences and can be exported to CSV for further analysis

### Inference with Video Support

For direct inference on videos, individual frames, or mixed folders, use `detector.ipynb`:
1. Configure input type (`'video'`, `'frames'`, or `'folder'`) and path
2. Set output FPS and video path
3. Run the notebook to generate annotated videos with labels and confidences
4. Review center coordinates and distances in the output CSV

## File Structure

```
Animal-Detector/
├── detector_trainer.ipynb    # Main training and inference notebook with ensemble
├── detector.ipynb            # Dedicated inference notebook for video processing
├── yolov8n.pt                 # Pre-trained YOLOv8 nano model
├── runs/                      # Training outputs
│   ├── detect/
│   │   └── train_custom/      # YOLO training results
│   │       ├── weights/       # YOLO model checkpoints
│   │       │   ├── best.pt    # Best YOLO weights
│   │       │   └── last.pt    # Last epoch YOLO weights
│   │       ├── results.csv    # YOLO training metrics
│   │       ├── results.png    # YOLO training curves
│   │       └── *.jpg          # YOLO validation predictions
│   └── classify/              # Classification model outputs
│       ├── alexnet_best.pt    # Best AlexNet weights
│       └── inception_best.pt  # Best Inception weights
├── reports/                   # Power BI report(s)
│   └── Animal-Detector.pbix   # Project report
├── outputs/                   # Example outputs
│   └── center_coordinates_example.csv  # Sample run of center coordinates
└── README.md                  # This file
```

## Expected Output

After training, you'll find:
- **YOLO model weights**: `runs/detect/train*/weights/best.pt`
- **Classification model weights**: `runs/classify/alexnet_best.pt`, `runs/classify/inception_best.pt`
- **Training metrics**: Loss curves, precision, recall, mAP scores for YOLO; accuracy/loss for classifiers
- **Validation images**: YOLO predictions on validation set
- **Confusion matrix**: YOLO model performance visualization
- **Ensemble inference results**: Per-model confidence scores in CSV exports
- **Power BI report**: `reports/Animal-Detector.pbix`
- **Example center coordinates**: `outputs/center_coordinates_example.csv` with ensemble scores

## Tips and Best Practices

- **Start small**: Begin with 25-50 frames per video to test your workflow
- **Diverse data**: Include various lighting conditions, angles, and backgrounds
- **Quality over quantity**: Well-annotated data is more valuable than large poorly-labeled datasets
- **Model selection**:
  - `yolov8n.pt`: Fastest, good for testing
  - `yolov8s.pt`: Small, balanced speed/accuracy
  - `yolov8m.pt`: Medium, better accuracy
  - `yolov8l.pt`: Large, high accuracy but slower
- **Ensemble threshold**: Adjust `ensemble_threshold` (default 0.5) based on your precision/recall needs; higher values reduce false positives
- **Training duration**: Start with 50-100 epochs for YOLO, 15 epochs for classifiers, and adjust based on validation metrics

## Troubleshooting

- **Out of memory**: Reduce batch size or use a smaller model (yolov8n); for classifiers, reduce batch size or use CPU if GPU memory is limited
- **Poor detection**: Add more diverse training data, train for more epochs, or adjust ensemble threshold
- **Overfitting**: Reduce epochs, add data augmentation, or increase regularization
- **Import errors**: Ensure all dependencies are installed with `pip install ultralytics opencv-python torch torchvision jupyter matplotlib`
- **Ensemble not working**: Check that all three models (YOLO, AlexNet, Inception) are trained and loaded correctly

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is open source and available under the MIT License.

## Acknowledgments

- Built with [Ultralytics YOLOv8](https://github.com/ultralytics/ultralytics)

## Contact

For questions or issues, please open an issue on GitHub.
