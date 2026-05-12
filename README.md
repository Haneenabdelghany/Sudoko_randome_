# Sudoku Solver Notebook (Computer Vision + KNN + Backtracking + SQLite)

## Overview

This project implements a complete end-to-end **Sudoku solving system** that operates on real-world images using a full computer vision and AI pipeline. It combines:

- Classical **Computer Vision (OpenCV)** for board detection and transformation  
- A **K-Nearest Neighbours (KNN)** classifier for digit recognition  
- A **backtracking search algorithm** with heuristics for solving Sudoku  
- A persistent **SQLite database layer** for logging and analytics  
- A full **visual overlay system** to render solutions back onto the original image  

The system is designed as a modular Jupyter Notebook with 20 structured cells, each representing a stage in the pipeline.

---
## Pipeline Architecture

Camera Image
↓
Preprocessing (grayscale, blur, threshold, dilation)
↓
Board Detection (contours + polygon approximation)
↓
Perspective Warp (homography transformation)
↓
Digit Recognition (KNN classifier)
↓
Grid Construction (9×9 matrix)
↓
Sudoku Solver (Backtracking + MCV heuristic)
↓
Solution Validation
↓
Overlay on Original Image (inverse warp)
↓
SQLite Storage (puzzles, logs, results)


---

## Key Technologies

### Computer Vision
- OpenCV (`cv2`)
- Contour detection
- Adaptive thresholding
- Perspective transformation (homography)

### Machine Learning
- K-Nearest Neighbours (KNN)
- Synthetic dataset generation for digits
- Feature extraction (20×20 grayscale flattening)

### Algorithmic Core
- Backtracking search
- Most Constrained Variable (MCV) heuristic
- Constraint propagation (row, column, subgrid elimination)

### Data Storage
- SQLite relational database
- JSON serialization for grids
- Base64 encoding for images

---

## Project Structure (Notebook Cells)

### 1. Environment Setup
Imports required libraries:
- OpenCV for image processing
- NumPy for matrix operations
- Scikit-learn for KNN
- SQLite for persistence
- Matplotlib for visualization

---

### 2. Database Layer
Creates a structured database with tables:

| Table | Purpose |
|------|--------|
| puzzles | Stores raw Sudoku grids |
| solutions | Stores solved grids + timing |
| detection_logs | Stores CV pipeline metadata |
| result_images | Stores final annotated images |

Key features:
- WAL mode for concurrency
- Foreign key enforcement
- JSON-based grid storage

---

### 3. Image Preprocessing
Transforms raw images into binary representations:

Steps:
1. Convert to grayscale  
2. Apply Gaussian blur  
3. Adaptive thresholding (handles uneven lighting)  
4. Morphological dilation (strengthens grid lines)  

Output:
- Clean binary image suitable for contour detection

---

### 4. Board Detection
Detects Sudoku grid using:

- `findContours()` → extracts shapes
- `approxPolyDP()` → approximates contours to polygons
- selects largest quadrilateral

Then:
- Computes confidence score based on area ratio
- Orders corners (TL, TR, BR, BL)

---

### 5. Perspective Transformation
Uses homography:

- Maps detected quadrilateral → perfect 450×450 square
- Ensures top-down aligned Sudoku grid
- Stores inverse transform for later overlay

Mathematical basis:
\[
(x', y', 1)^T = H (x, y, 1)^T
\]

---

### 6. Digit Recognition (KNN)

#### Dataset Generation
Synthetic digits generated using:
- Multiple fonts
- Scale variations
- Thickness variation
- Positional noise
- Blur augmentation

#### Feature Pipeline
Each digit:
1. Cropped
2. Thresholded
3. Resized to 20×20
4. Flattened into 400-dimensional vector

#### Classifier
- KNN (k = 5)
- Distance-weighted voting
- Euclidean distance metric

---

### 7. Grid Extraction

- Split warped image into 81 cells (9×9)
- Extract each cell (50×50 pixels)
- Apply Otsu thresholding per cell
- Detect empty vs digit cells using pixel density
- Classify digits using trained KNN model

Output:
- 9×9 integer matrix

---

### 8. Sudoku Solver

Core solving algorithm:

#### Backtracking Search
- Recursively assigns values
- Reverts invalid assignments

#### Optimisation: MCV Heuristic
Chooses the cell with:
> minimum number of valid candidates

This dramatically reduces search space.

#### Constraint Rules
For each cell:
- Row uniqueness
- Column uniqueness
- 3×3 subgrid uniqueness

Worst-case complexity:
- O(9^N), heavily reduced by heuristics

---

### 9. Solution Validation

Ensures correctness by verifying:
- Every row contains 1–9
- Every column contains 1–9
- Every subgrid contains 1–9

Uses set comparison for O(1) validation per region.

---

### 10. Overlay System

Purpose:
Render solution back onto original image.

Steps:
1. Draw solved digits on blank 450×450 canvas
2. Apply inverse homography
3. Merge with original image using mask
4. Preserve original clues (only fill empty cells)

Output:
- Realistic solved Sudoku image

---

### 11. Database Logging

Stores:
- Puzzle input
- Solution output
- Solve time (ms)
- Validity flag
- Detection metadata
- Encoded result image

Supports:
- Full audit trail
- Performance analysis
- Batch evaluation

---

### 12. Batch Processing

Processes multiple images:

- Runs full pipeline per image
- Handles failures gracefully
- Aggregates results
- Outputs success statistics

---

## Performance Characteristics

| Component | Complexity | Notes |
|----------|-----------|------|
| Digit recognition | O(81 × KNN) | Fast due to small dataset |
| Solver | exponential worst-case | Reduced via MCV heuristic |
| CV pipeline | O(n pixels) | Linear in image size |

Typical solve time:
- ~10–50 ms for solver
- ~200–500 ms full pipeline

---

## Key Strengths

- Fully automated end-to-end system
- Robust to lighting variations
- Handles perspective distortion
- Interpretable classical ML approach (no deep learning required)
- Persistent logging system for analytics
- Modular notebook design

---

## Limitations

- KNN digit classifier is sensitive to extreme handwriting styles
- No guarantee of unique puzzle generation
- Performance depends on image quality
- Classical CV may fail under severe distortion

---

## Possible Improvements

- Replace KNN with CNN (e.g., LeNet or lightweight ResNet)
- Add constraint propagation (forward checking)
- Improve board detection using deep learning segmentation
- Add real-time webcam integration
- Enforce Sudoku uniqueness during generation
- Optimize solver using DLX (Dancing Links Algorithm)

---

## Conclusion

This project demonstrates a complete integration of:
- Computer Vision
- Classical Machine Learning
- Graph Search Algorithms
- Database Systems

It is a strong example of how traditional AI techniques can still form a powerful, interpretable, and efficient pipeline for structured visual reasoning problems like Sudoku.

---
