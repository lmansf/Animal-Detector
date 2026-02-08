# Animal Detector

A YOLOv8-based animal detection system that extracts frames from videos, trains a custom object detector, and tracks animal movements. This project provides a complete workflow from video processing to motion tracking.

## Overview

This project uses YOLOv8 (You Only Look Once) to detect and track animals in video footage. The notebook-based workflow guides you through:
- Extracting frames from video files
- Preparing datasets for training
- Training a custom YOLOv8 detector
- Running inference on new images
- Tracking animal positions and measuring distances traveled

## Features

- 🎥 **Video Frame Extraction**: Automatically extract frames from multiple videos
- 🏷️ **YOLO Dataset Preparation**: Organize labeled data into train/validation splits
- 🤖 **Custom Model Training**: Train YOLOv8 on your own animal dataset
- 🎯 **Object Detection**: Detect animals in images with bounding boxes
- 📍 **Center Point Tracking**: Calculate pixel coordinates of detected objects
- 📏 **Distance Measurement**: Track movement across video frames

## Prerequisites

- Python 3.7 or higher
- Jupyter Notebook or JupyterLab
- Basic understanding of object detection and YOLO format

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
- [LabelImg](https://github.com/heartexlabs/labelImg)
- [Roboflow](https://roboflow.com/)
- [CVAT](https://github.com/opencv/cvat)

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

## File Structure

```
Animal-Detector/
├── detector_trainer.ipynb    # Main training and inference notebook
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
└── README.md                  # This file
```

## Expected Output

After training, you'll find:
- **Model weights**: `runs/detect/train*/weights/best.pt`
- **Training metrics**: Loss curves, precision, recall, mAP scores
- **Validation images**: Predictions on validation set
- **Confusion matrix**: Model performance visualization

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
- Powered by [OpenCV](https://opencv.org/) for video processing

## Contact

For questions or issues, please open an issue on GitHub.
