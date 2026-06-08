# 🚗 Self-Driving Car: Advanced Lane Detection Pipeline

![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![OpenCV](https://img.shields.io/badge/OpenCV-Computer_Vision-green)
![NumPy](https://img.shields.io/badge/NumPy-Data_Processing-lightblue)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange)

A Computer Vision pipeline designed for autonomous vehicle perception. This repository contains the algorithms and mathematical models required to process dashcam video feeds, isolate road lanes in varying lighting conditions, mathematically model the road's curvature, and predict the vehicle's required steering direction (straight, left, or right) in real-time.

## 🎯 Project Overview: What Are We Trying to Achieve?

For a self-driving car to navigate safely, it must perfectly understand its position within the lane and anticipate the geometry of the road ahead. The goal of this project is to build a highly accurate **Perception Module** that ignores irrelevant background noise (sky, trees, other cars) and isolates lane markings. 

By calculating the mathematical curve of these lanes, the system outputs actionable navigation commands—acting as the "eyes" and basic "steering logic" of an autonomous vehicle.

## 🧠 In-Depth Analysis: How Are We Achieving It?

This project handles the extreme variability of real-world driving (shadows, faded lines, glare) through a robust, multi-stage computer vision pipeline. We evolved from a basic Hough Transform approach to an Advanced Sliding Window algorithm to handle sharp curves.

### 1. Advanced Preprocessing & Color Space Tuning
Raw RGB video is highly susceptible to lighting changes. 
* **Histogram Equalization:** We convert the image to YUV color space and equalize the Y (luminance) channel to balance harsh shadows and bright glares.
* **HSV Masking:** We convert the image to HSV (Hue, Saturation, Value) to create a strict color mask. By defining a specific lower `[2, 60, 190]` and upper `[80, 255, 255]` boundary, we aggressively isolate yellow and white lane markers while suppressing the grey asphalt and green scenery.
* **Morphological Filtering:** Min/Max filters and Median Blurs are applied to eliminate "salt and pepper" noise (like small rocks or reflections).

### 2. Perspective Transformation (The "Bird's-Eye View")
Standard camera angles distort parallel lines, making them converge at the horizon. 
* We define a Region of Interest (ROI) using a polygon that covers the immediate road.
* Using `cv2.getPerspectiveTransform`, we mathematically warp the image into a top-down, 2D "Bird's-Eye" view. In this view, lane lines appear perfectly parallel, allowing us to accurately measure their true curvature.

### 3. Histogram Peak & Sliding Window Search
Once we have a top-down binary mask of the lanes, we need to track them from the bottom of the screen to the top.
* **Base Detection:** We calculate a histogram across the bottom half of the image. The two highest peaks of white pixels indicate the starting *x-coordinates* of the left and right lanes.
* **Sliding Windows:** We stack rectangular "windows" (100x50 pixels) moving upwards. Inside each window, we use `cv2.findContours` and image moments (`cv2.moments`) to find the center of mass of the lane pixels, dynamically adjusting the window left or right to track the curve of the road.

### 4. Curve Fitting & Turn Prediction
With the center points of the lane identified across multiple windows, we map the geometry of the road.
* **Polynomial Fitting:** We use `np.polyfit` to fit a 2nd-degree polynomial curve through the detected points.
* **Steering Output:** By calculating the slope of this curve, we determine the turn angle. If the angle exceeds a `TURN_THRESHOLD` of 5.0 degrees, the system predicts a "Right turn" or "Left turn" along with a dynamic confidence score based on the number of successfully tracked windows.

## 📂 Repository Structure

| File/Folder | Description | 
| ----- | ----- | 
| `notebook/SelfDrivingCar.ipynb` | The core Jupyter Notebook containing the entire experimental pipeline, from basic Hough transforms to the advanced sliding window approach. | 
| `DIP Project Videos/` | Directory containing the input `.mp4` dashcam footage and `.png` test frames used to calibrate the algorithms. | 
| `.idea/` | IDE configuration files (can be safely ignored by non-PyCharm/IntelliJ users). | 
| `.gitignore` | Standard Git ignore file to keep the repository clean of temporary artifacts. | 

## ⚙️ Complete Setup & Installation Guide

### Step 1: Prerequisites
Ensure you have **Python 3.8+** installed on your machine. You will also need Jupyter Notebook or JupyterLab to interact with the `.ipynb` file.

### Step 2: Clone the Repository
Open your terminal and clone the repository to your local machine:

```bash
git clone <your-repository-url>
cd dip-p-self-driving-car
```

### Step 3: Create a Virtual Environment (Recommended)
Isolate your project dependencies to avoid conflicts:

```bash
# Windows
python -m venv venv
venv\Scripts\activate

# macOS/Linux
python3 -m venv venv
source venv/bin/activate
```

### Step 4: Install Dependencies
This project relies heavily on OpenCV and NumPy. Install the required computer vision and data processing libraries:

```bash
pip install opencv-python numpy matplotlib jupyterlab
```

### Step 5: Verify Video Paths
Ensure your video files (e.g., `PXL_20250325_045117252.TS.mp4`) are correctly placed inside the `DIP Project Videos/` folder. The Jupyter Notebook uses relative paths `../DIP Project Videos/...` to access these files.

## 🚀 Usage Guide

### Running the Notebook
Because this project is built as a Jupyter Notebook, you will run it interactively. This is excellent for visualizing how the image transforms at every single step.

1. Launch Jupyter from your terminal:
```bash
jupyter notebook
```
2. In the browser interface, navigate to the `notebook/` folder and open `SelfDrivingCar.ipynb`.
3. Run the cells sequentially (Shift + Enter). 
4. The final cells will open an OpenCV GUI window playing the video feed with real-time lane tracking overlays and turn predictions!

*(To stop the video playback at any time, press the `q` key on your keyboard while the video window is focused).*

## 🛠️ Troubleshooting

* **`cv2.error: (-215:Assertion failed) !empty()`:** This means OpenCV cannot find your video or image file. Double-check that the file paths in the `cv2.VideoCapture()` and `cv2.imread()` functions exactly match the locations of your files relative to the notebook.
* **Kernel Crashes on Video Exit:** If the Jupyter kernel crashes when you close the OpenCV window, make sure you are clicking on the video window and pressing `q` to trigger the `cap.release()` and `cv2.destroyAllWindows()` commands gracefully, rather than clicking the "X" on the window.
* **No lanes detected in your own video:** The `tl, bl, tr, br` Region of Interest coordinates and the `lower_bound`/`upper_bound` HSV values are highly calibrated to the specific dashcam position and lighting of the provided videos. If you use your own footage, you must adjust these hardcoded values to match your camera's perspective.

---
*Developed with ❤️ using OpenCV and Python.*
