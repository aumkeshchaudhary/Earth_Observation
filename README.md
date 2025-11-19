# 🌍 Delhi Airshed – Earth Observation Pipeline  
### AI-Based Land Use Classification using Sentinel-2 & ESA WorldCover

This repository implements a complete geospatial machine-learning pipeline for analyzing the Delhi Airshed region. The goal is to classify land-use patterns and evaluate pollution-related structures using Sentinel-2 imagery and ESA WorldCover 2021 land-cover data.

---

## 1. Project Overview

This project covers:

- Spatial gridding of Delhi-NCR  
- Satellite-basemap visualization  
- Filtering Sentinel-2 image patches by geospatial location
- Extracting land-cover patches from ESA WorldCover 
- Label assignment via mode of land-cover pixels 
- Train/test dataset construction
- Training a ResNet18 classifier
- Computing F1-scores, confusion matrices, and prediction visualizations

All code is contained in:  
**`Earth_Observation.ipynb`**

---

## 2. Spatial Reasoning & Data Filtering (Q1)

### 2.1 Delhi-NCR Shapefile + Grid Overlay
- Loaded shapefile using `geopandas`
- Reprojected to EPSG:32644 (UTM zone) because a 60×60 km grid requires a metric CRS
- Constructed a uniform **60 km × 60 km** grid over the polygon
- Visualized the grid using matplotlib

### 2.2 Satellite Basemap Overlay
- Used `geemap.Map()`
- Added `"SATELLITE"` basemap
- Reprojected grid back to EPSG:4326 for display on geemap
  (grid = grid.to_crs(4326))

### 2.3 Marking Grid Corners & Centers
- Extracted the four corners (minx,miny), (maxx,miny), (maxx,maxy), (minx,maxy)
- Computed centroid using cell.centroid
- Plotted corners + centroid on the basemap

### 2.4 Filtering Sentinel-2 Image Patches
Each PNG file has center coordinates embedded in filename:
 image_lat_lon.png (lat, lon in EPSG:4326)

Steps:
- Parsed (lat, lon)
- Converted to geometry and reprojected to EPSG:32644
- Checked if point lies inside any grid polygon
- Saved filtered images to images_filtered.csv

### 2.5 Image Count
- Printed number of images before filtering
- Printed number after filtering

---

## 3. Label Construction & Dataset Preparation (Q2)

### 3.1 Extracting 128×128 Land-Cover Patch
For each Sentinel-2 RGB patch:
- Loaded land_cover.tif using rasterio
- Converted (lon, lat) → raster pixel indices using dataset.index()
- Extracted a 128×128 window centered on that pixel
- Handled raster boundaries & nodata conditions

### 3.2 Mode-Based Label Assignment
A patch typically contains many classes:

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
- Coordinates outside raster bounds → skipped
- Patches with >30% nodata → removed
- Mixed class dominance → strong mode chosen
- Tie situations → lowest ESA class code selected


### 3.5 Train-Test Split (60/40)

Used:

     train_test_split(..., test_size=0.4, stratify=labels, random_state=42)
 If a class had <2 samples, stratification fails. In these cases, a fallback non-stratified split with fixed seed was used.

  
### 3.6 Class Distribution Visualization
- Generated bar plot of class counts
- Discussed class imbalance based on extracted labels.

---

## 4. Model Training & Evaluation (Q3)

### 4.1 CNN Model Training
- Used **ResNet18**
- Image input size: 128×128 RGB
- Pretrained ImageNet weights
- Modified final FC to match number of classes
- Loss: CrossEntropyLoss
- Optimizer: Adam

### 4.2 Custom F1-Score Implementation
Manually calculated:
- TP, FP, FN  
- Macro F1
- Weighted F1  

### 4.3 torchmetrics.F1Score
- Compared with manual implementation  
- Differences only due to rounding & averaging behaviour

### 4.4 Confusion Matrix
- Computed using sklearn.metrics.confusion_matrix
- Row-normalized heatmap saved to confusion_matrix.png

### 4.5 Correct & Incorrect Prediction Visualization
Saved:
- 5 correctly classified images 
- 5 misclassified images
Each contains true + predicted labels.

---

## 5. Repository Structure

      Earth-Observation-Delhi-Airshed
      ├── archive (15)/    
      |    ├── rgb/
      |    |     ├── 28.2056_76.8558.png
      |    |     ├── 28.2056_76.8646.png
      |    |     └── ........
      |    ├── delhi_airshed.geojson
      |    ├── delhi_ncr_region.geojson
      |    └── worldcover_bbox_delhi_ncr_2021.tif
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
      │       └── 
      ├── Earth_Observation.ipynb       # Complete pipeline (Q1–Q3)
      └── README.md




---

## 6. Technologies Used

- Python
- geopandas
- rasterio
- shapely
- pyproj
- numpy / pandas
- matplotlib
- geemap
- PyTorch
- torchmetrics

---

## 7. Final Remarks

This repository demonstrates a complete, end-to-end Earth Observation ML pipeline using real satellite data. It integrates spatial operations, raster label extraction, dataset building, CNN training, and evaluation.

---
