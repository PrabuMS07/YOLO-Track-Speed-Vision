
# 🚗 YOLOv8 Speed Detection System 💨

This project utilizes the YOLOv8 object detection model combined with the Norfair tracking library to estimate the speed of detected objects (primarily intended for vehicles) as they cross two predefined lines in a video feed.

## ✨ Features

*   **Real-Time Object Detection:** Uses YOLOv8 (`yolov8l.pt` model) to detect objects in a video stream.
*   **Object Tracking:** Employs Norfair to assign unique IDs and track detected objects across frames.
*   **Line Crossing Detection:** Monitors when tracked objects cross two user-defined horizontal lines.
*   **Speed Estimation:** Calculates the approximate speed of an object based on the time taken to travel between the two lines and a predefined real-world distance.
*   **On-Screen Display:** Visualizes detected objects, tracking IDs, the detection lines, and the calculated speed directly on the video feed.
*   **Console Output:** Prints detection events and calculated speeds to the console.

## ⚙️ How It Works

1.  **Initialization:** Loads the YOLOv8 model and initializes the Norfair tracker and video capture (from webcam index 0).
2.  **Line Definition:** Two horizontal red lines (`red_line_1_y`, `red_line_2_y`) are defined at specific y-coordinates on the frame.
3.  **Frame Processing Loop:**
    *   Reads a frame from the video source.
    *   Performs object detection using YOLOv8.
    *   Converts YOLO detections into Norfair `Detection` objects (using the center point).
    *   Updates the Norfair tracker with the current detections to get tracked object states (ID and estimated position).
4.  **Line Crossing Logic:**
    *   For each tracked object:
        *   If it crosses the *first* line (`red_line_1_y`) and hasn't been recorded yet, its ID and the current frame number are stored.
        *   If it subsequently crosses the *second* line (`red_line_2_y`) and its entry frame is known, the time difference (calculated using frame difference and video FPS) is determined.
5.  **Speed Calculation:**
    *   Using the known `real_distance_meters` between the two lines in the real world and the calculated `time_diff`, the speed is calculated: `Speed = Distance / Time`.
    *   The speed is converted from meters per second (m/s) to kilometers per hour (km/h).
    *   The calculated speed is stored and displayed near the object.
6.  **Cleanup:** Once speed is calculated for an object, its initial frame record is removed to allow for re-detection if it loops back (though primarily designed for one-way traffic).
7.  **Visualization:** Draws the detection lines and overlays the calculated speed text onto the video frame.
8.  **Display & Exit:** Shows the processed frame in a window. Exits when 'q' is pressed.

## 🚀 Technologies Used

*   **Python 3.x**
*   **OpenCV (`opencv-python`)**: For video capture, image processing, drawing, and display.
*   **Ultralytics (`ultralytics`)**: For the YOLOv8 object detection model.
*   **Norfair (`norfair`)**: For real-time object tracking.
*   **NumPy**: For numerical operations.

## 🛠️ Setup & Installation

1.  **Clone the Repository:**
    ```bash
    git clone <your-repo-url>
    cd <your-repo-directory>
    ```
2.  **Set up Python Environment (Recommended):**
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows use `venv\Scripts\activate`
    ```

    *(Note: Ensure you have a compatible version of PyTorch installed, potentially with CUDA support if you plan to use `device="cuda"` as in the script. Check PyTorch and Ultralytics documentation for specific requirements.)*
3.  **YOLOv8 Model:** The script uses `yolov8l.pt`. The `ultralytics` library should download this automatically on first run if it's not present.

## ⚙️ Configuration

Before running, you **must** configure these critical parameters in the script based on your specific camera setup:

*   `real_distance_meters`: **Crucial.** Set this to the actual physical distance (in meters) between the points in the real world that correspond to the two red lines (`red_line_1_y`, `red_line_2_y`) in the camera's view. **Accurate calibration is key for accurate speed.**
*   `red_line_1_y`: The y-coordinate (pixel row) for the first detection line.
*   `red_line_2_y`: The y-coordinate (pixel row) for the second detection line. Adjust these based on your camera view to define the start and end points for speed measurement.
*   `cap = cv2.VideoCapture(0)`: Change `0` if your webcam has a different index or replace with a video file path (e.g., `cv2.VideoCapture("my_video.mp4")`).
*   `tracker = Tracker(...)`: Adjust `distance_threshold` if tracking is inaccurate (higher allows for more movement between frames, lower is stricter).
*   `model.predict(...)`: Adjust `conf=0.5` (confidence threshold) if you need to detect less certain objects or reduce noise. Change `device="cuda"` to `device="cpu"` if you don't have a compatible GPU setup.

## ▶️ Usage

1.  **Activate Environment:** `source venv/bin/activate` (if applicable).
2.  **Run the Script:**
    ```bash
    python your_script_name.py
    ```
    *(Replace `your_script_name.py` with the actual filename)*
3.  **Observe:** A window titled "Speed Detection" will open, showing the camera feed with detection lines, tracked objects, and calculated speeds. Console output will show line crossing events and speed calculations.
4.  **Exit:** Press 'q' while the video window is active to stop the script.


## 💡 Potential Future Improvements

*   **Camera Calibration & Perspective Correction:** Use OpenCV's camera calibration and `getPerspectiveTransform`/`warpPerspective` to map pixel coordinates to real-world coordinates for more accurate distance measurement, regardless of position in the frame.
*   **Multiple Lanes/Zones:** Define multiple pairs of lines or zones to handle different lanes of traffic.
*   **Direction Detection:** Determine the direction of travel and handle objects moving between line 2 and line 1.
*   **Averaging/Smoothing:** Calculate speed over more frames or average multiple readings for robustness.
*   **Data Logging:** Save calculated speeds, timestamps, and object IDs to a file (CSV, database).
*   **GUI:** Develop a more user-friendly interface using libraries like Tkinter, PyQt, or Kivy.
*   **Robust FPS Calculation:** Implement manual FPS calculation over intervals if `CAP_PROP_FPS` is unreliable.
