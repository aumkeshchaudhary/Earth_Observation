# 🌍 Delhi Airshed – Earth Observation Pipeline  
### AI-Based Land Use Classification using Sentinel-2 & ESA WorldCover

This repository implements a complete geospatial machine-learning pipeline for analyzing the Delhi Airshed region. The goal is to classify land-use patterns and evaluate pollution-related structures using Sentinel-2 imagery and ESA WorldCover 2021 land-cover data.

---

## 1. Project Overview

This project covers:

- Spatial gridding of Delhi-NCR  
- Satellite-basemap visualization  
- Image filtering using geospatial coordinates  
- Land-cover label extraction (from ESA WorldCover raster)  
- Dataset creation for supervised learning  
- Training a ResNet18 classifier  
- Evaluation using F1-scores and confusion matrix  
- Visualizing correct & incorrect predictions  

All code is contained in:  
**`Earth_Observation.ipynb`**

---

## 2. Spatial Reasoning & Data Filtering (Q1)

### 2.1 Delhi-NCR Shapefile + Grid Overlay
- Loaded shapefile using `geopandas`
- Transformed CRS to **EPSG:32644**
- Created a **60 × 60 km uniform grid**
- Visualized using `matplotlib`

### 2.2 Satellite Basemap Overlay
- Used `geemap.Map()`
- Added `"SATELLITE"` basemap
- Overlaid the grid with transparency

### 2.3 Marking Grid Corners & Centers
- Computed:
  - Four corners
  - Centroid of each grid cell
- Plotted each on the basemap

### 2.4 Filtering Image Patches
- PNG filenames contain `(lat, lon)`
- Checked if each point falls inside any grid polygon
- Stored filtered images into `images_filtered.csv`

### 2.5 Image Count
- Printed number of images before and after filtering.

---

## 3. Label Construction & Dataset Preparation (Q2)

### 3.1 Extracting Land-Cover Patch
For each Sentinel-2 image:
- Read `land_cover.tif` using `rasterio`
- Converted image center coordinate to raster index
- Extracted a **128 × 128** land-cover patch

### 3.2 Mode-Based Label Assignment
Example pixel counts:
- Built-up (50) → 8000 pixels  
- Cropland (40) → 3000 pixels  
- Tree Cover (10) → 512 pixels  

Final class = **Built-up** (majority class).

### 3.3 ESA → Standardized Label Map
| ESA Code | Class Name |
|----------|------------|
| 10 | Tree Cover |
| 20 | Shrubland |
| 30 | Grassland |
| 40 | Cropland |
| 50 | Built-up |
| 60 | Bare Sparse Vegetation |
| 70 | Snow & Ice |
| 80 | Permanent Water |
| 90 | Herbaceous Wetland |
| 95 | Mangroves |
| 100 | Moss / Lichen |

### 3.4 Handling Edge Cases
- **No-data pixels** → sample skipped  
- **Coordinates outside raster** → skipped  
- **Mixed class dominance** → used strongest mode  
- **Heavily noisy patches** → removed from dataset  

### 3.5 Train-Test Split (60/40)
Performed using `train_test_split` with stratification.

### 3.6 Class Distribution Visualization
Generated a bar plot showing imbalance across classes.

---

## 4. Model Training & Evaluation (Q3)

### 4.1 CNN Model Training
- Used **ResNet18**
- Trained on 128×128 RGB images
- Optimizer: Adam  
- Loss: CrossEntropyLoss  

### 4.2 Custom F1-Score Implementation
Manually calculated:
- TP, FP, FN  
- Macro and weighted F1 scores  

### 4.3 torchmetrics.F1Score
- Compared with manual implementation  
- Discussed minor differences in rounding

### 4.4 Confusion Matrix
Generated using:
- `sklearn.metrics.confusion_matrix`
- Normalized heatmap

### 4.5 Correct & Incorrect Prediction Visualization
Plotted:
- 5 correctly classified images  
- 5 misclassified images  
- Included true + predicted labels  

---

## 5. Repository Structure

      Earth-Observation-Delhi-Airshed
      ├── Earth_Observation.ipynb       # Complete pipeline (Q1–Q3)
      ├── outputs/
      │   ├── images_filtered.csv
      │   ├── train_split.csv
      │   ├── test_split.csv
      │   ├── model_resnet18.pth
      │   ├── confusion_matrix.png
      │   ├── class_distribution.png
      │   └── predictions_visualization/
      │       ├── correct_0_true_X_pred_X.png
      │       ├── correct_1_true_X_pred_X.png
      │       ├── incorrect_0_true_X_pred_Y.png
      │       └── ...
      └── README.md




---

## 6. Technologies Used

- Python  
- geopandas  
- rasterio  
- shapely  
- pyproj  
- numpy / pandas  
- PyTorch  
- torchmetrics  
- matplotlib  
- geemap  

---

## 7. Final Remarks

This repository demonstrates a complete end-to-end Earth Observation ML pipeline using real geospatial datasets.  
The work includes spatial processing, raster extraction, dataset creation, CNN training, and evaluation.

---
