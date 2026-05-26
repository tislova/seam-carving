# Content-Aware Image Resizing — Seam Carving

A from-scratch implementation of **seam carving**, the content-aware image 
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

This project implements the full pipeline by hand using NumPy: energy 
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