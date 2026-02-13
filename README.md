# Animal Detector

A YOLOv8-based animal detection system that extracts frames from videos, trains a custom object detector, and tracks animal movements. This project provides a complete workflow from video processing to motion tracking.

## Overview

This project uses YOLOv8 (You Only Look Once) to detect and track animals in video footage. The notebook-based workflow guides you through:
- Extracting frames from video files
- Preparing datasets for training
- Training a custom YOLOv8 detector
- Running inference on new images
- Tracking animal positions and measuring distances traveled
- Reviewing results in a Power BI report
- Inspecting an example run of center-coordinate outputs

## Recent Changes

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
| Train or re-train a model | `detector_trainer.ipynb` | Step 4 |
| Validate model and inspect training metrics | `detector_trainer.ipynb` | Step 4 |
| Run inference on a single image/folder (trainer flow) | `detector_trainer.ipynb` | Step 5 |
| Measure movement distance across ordered frames | `detector_trainer.ipynb` | Step 6 |
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
   pip install ultralytics opencv-python jupyter
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

### Step 4: Prepare Train/Validation Dataset

1. Place all labeled frames and their corresponding `.txt` label files in one folder
2. Set `labeled_root` to that folder in the notebook
3. Run the preparation cell to automatically split data into:
   - `images/train` and `labels/train` (80% of data)
   - `images/val` and `labels/val` (20% of data)

### Step 5: Train the Detector

1. Choose a base model (default: `yolov8n.pt` - nano model for faster training)
2. Configure training parameters (epochs, image size, etc.)
3. Run the training cell
4. Monitor training progress and metrics
5. Best model weights are saved in `runs/detect/train*/weights/best.pt`

### Step 6: Run Inference

Test your trained model:
1. Select an image for testing
2. Load your trained weights
3. Run detection to see bounding boxes and confidence scores

### Step 7: Get Center Point Coordinates

Extract the center pixel coordinates `(x, y)` of each detected animal for position tracking.

### Step 8: Measure Distance Traveled

Track animal movement across video frames:
1. Provide a folder of ordered frames (from a video)
2. Run detection on all frames
3. Calculate distances between detected positions
4. Results can be exported to CSV for further analysis

### Inference with Video Support

For direct inference on videos, individual frames, or mixed folders, use `detector.ipynb`:
1. Configure input type (`'video'`, `'frames'`, or `'folder'`) and path
2. Set output FPS and video path
3. Run the notebook to generate annotated videos with labels and confidences
4. Review center coordinates and distances in the output CSV

## File Structure

```
Animal-Detector/
├── detector_trainer.ipynb    # Main training and inference notebook
├── detector.ipynb            # Dedicated inference notebook for video processing
├── yolov8n.pt                 # Pre-trained YOLOv8 nano model
├── runs/                      # Training outputs
│   └── detect/
│       └── train_custom/      # Training results
│           ├── weights/       # Model checkpoints
│           │   ├── best.pt    # Best model weights
│           │   └── last.pt    # Last epoch weights
│           ├── results.csv    # Training metrics
│           ├── results.png    # Training curves
│           └── *.jpg          # Validation predictions
├── reports/                   # Power BI report(s)
│   └── Animal-Detector.pbix   # Project report
├── outputs/                   # Example outputs
│   └── center_coordinates_example.csv  # Sample run of center coordinates
└── README.md                  # This file
```

## Expected Output

After training, you'll find:
- **Model weights**: `runs/detect/train*/weights/best.pt`
- **Training metrics**: Loss curves, precision, recall, mAP scores
- **Validation images**: Predictions on validation set
- **Confusion matrix**: Model performance visualization
- **Power BI report**: `reports/Animal-Detector.pbix`
- **Example center coordinates**: `outputs/center_coordinates_example.csv`

## Tips and Best Practices

- **Start small**: Begin with 25-50 frames per video to test your workflow
- **Diverse data**: Include various lighting conditions, angles, and backgrounds
- **Quality over quantity**: Well-annotated data is more valuable than large poorly-labeled datasets
- **Model selection**:
  - `yolov8n.pt`: Fastest, good for testing
  - `yolov8s.pt`: Small, balanced speed/accuracy
  - `yolov8m.pt`: Medium, better accuracy
  - `yolov8l.pt`: Large, high accuracy but slower
- **Training duration**: Start with 50-100 epochs and adjust based on validation metrics

## Troubleshooting

- **Out of memory**: Reduce batch size or use a smaller model (yolov8n)
- **Poor detection**: Add more diverse training data or train for more epochs
- **Overfitting**: Reduce epochs or add data augmentation
- **Import errors**: Ensure all dependencies are installed with `pip install ultralytics opencv-python jupyter`

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is open source and available under the MIT License.

## Acknowledgments

- Built with [Ultralytics YOLOv8](https://github.com/ultralytics/ultralytics)

## Contact

For questions or issues, please open an issue on GitHub.
