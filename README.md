# Content-Aware Image Resizing — Seam Carving

Aт implementation of **seam carving**, the content-aware image 
resizing algorithm introduced by Avidan & Shamir (2007). Unlike standard 
resizing, which scales every pixel uniformly and distorts the subject, seam 
carving removes low-energy "seams" so the important content of an image keeps 
its proportions while the dimensions shrink.

![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![NumPy](https://img.shields.io/badge/numpy-013243?style=for-the-badge&logo=numpy&logoColor=white)
![OpenCV](https://img.shields.io/badge/opencv-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white)
![Matplotlib](https://img.shields.io/badge/matplotlib-11557C?style=for-the-badge&logo=python&logoColor=white)

## Overview

Standard image resizing (e.g. `cv2.resize`) squashes or stretches every pixel 
equally. If you shrink a photo's width, people get thinner, cars get narrower, 
buildings lean. Seam carving solves this by repeatedly finding and removing the 
single connected path of pixels, a *seam*, that carries the least visual 
information, leaving the high-energy subject untouched.

This project implements the full pipeline using NumPy: energy 
computation, dynamic-programming cost maps, optimal seam backtracking, and 
iterative pixel removal, then compares the results against standard resizing 
on several test images.

## How It Works

The algorithm runs in four stages, each implemented as its own function:

1. **Energy map** — Convert the image to grayscale and measure how much each 
   pixel "matters" using image gradients. High-energy pixels sit on edges and 
   detail; low-energy pixels sit in smooth regions like sky or water.
2. **Cumulative energy map** — Using dynamic programming, build a map where 
   each cell holds the minimum total energy of any seam reaching that pixel 
   from one side of the image. Vertical seams accumulate top-to-bottom; 
   horizontal seams accumulate left-to-right.
3. **Seam backtracking** — Start at the lowest-energy pixel on the final 
   row/column and walk backwards, at each step choosing the cheapest of the 
   (up to) three connected neighbors. This yields the single optimal seam.
4. **Seam removal** — Delete the seam by slicing and concatenating around it, 
   then recompute the energy map and repeat until the target size is reached.

## Functions

- **`energy_img`** — Baseline energy via `np.gradient` (combined x/y gradient 
  magnitude on the grayscale image).
- **`cumulative_min_energy_map`** — Dynamic-programming cost map for either 
  `"VERTICAL"` or `"HORIZONTAL"` seams, with explicit handling of the border 
  columns/rows where fewer neighbors are available.
- **`find_vertical_seam`** / **`find_horizontal_seam`** — Backtrack through the 
  cumulative map to recover the optimal seam as a vector of column (or row) 
  indices.
- **`view_seam`** — Overlay a found seam on the image in red for visualization.
- **`decrease_width`** / **`decrease_height`** — Remove one vertical/horizontal 
  seam and return the smaller image plus its recomputed energy map.

### Alternative Energy Functions

Part of the project explores how the choice of energy function changes which 
seams get removed. Three additional operators were implemented from scratch 
with manual 2D convolution (edge-padding + sliding 3×3 / 2×2 windows):

- **`energy_img_modified_lap`** — Laplacian filter `[[0,1,0],[1,-4,1],[0,1,0]]`, 
  which highlights edges more aggressively than the gradient method.
- **`energy_img_modified`** — Prewitt operator (separate x and y kernels).
- **`energy_img_roberts`** — Roberts cross operator (2×2 kernels).

## Results & Observations

The notebook runs the pipeline on several images and compares seam carving to 
standard resizing:

- **Energy & cumulative maps** are visualized to show low-energy regions (sky, 
  water) as dark paths and high-energy subjects (boats, buildings) as bright 
  barriers that seams route around.
- **Laplacian energy** does a noticeably better job preserving building edges 
  than the basic gradient, because it assigns higher energy to those edges so 
  seams avoid them.
- **Subject preservation** — On images like the car and the door, seam carving 
  removes only background ground/sky pixels and keeps the main object's 
  proportions, whereas standard resizing visibly distorts it.
- **Known failure case** — On an image dominated by vertical structures, seam 
  carving thins those objects rather than removing background, and standard 
  resizing actually produces the better result. This is documented as an honest 
  limitation of the technique.

## Technologies Used

- **Python**
- **NumPy** — all matrix operations, gradients, and manual convolution
- **OpenCV** (`cv2`) — image loading and the baseline `resize` comparison
- **Matplotlib** — image display, energy-map visualization, and result plots

## Running It

```bash
pip install numpy opencv-python matplotlib
```

Open `main.ipynb` in Jupyter or Google Colab and run the cells top to bottom. 
The input images (e.g. `inputSeamCarvingPrague.jpg`, `inputSeamCarvingBuilding.jpg`) 
need to be in the same directory as the notebook.