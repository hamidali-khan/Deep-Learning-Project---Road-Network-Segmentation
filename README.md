# Road Network Extraction from Aerial Imagery

---

## Project Overview
This project delivers a fully automated deep-learning pipeline that converts high-resolution aerial orthophotos into clean, topologically correct road centerline graphs—ready for GPS navigation, urban planning, or GIS updates. Building on the Deep ResUNet + B-snake approach of Munawar et al. (2023), we implement three successive segmentation models and smooth their outputs with a cubic active-contour (B-snake) post-processing step 

---

## Key Contributions
- **Baseline (Deliverable 3)**: Deep ResUNet with residual blocks and boundary-aware loss scheduling  
- **Deliverable 4**: Addition of CBAM attention modules and a lightweight ASPP context block  
- **Deliverable 5**: Nested U-Net++ with Attention Gates, deep supervision, and Focal Tversky + CosineAnnealingLR  
- **Post-Processing**: Cubic B-spline active-contour (“B-snake”) smoothing for topologically consistent centerlines

---

## Dataset & Preprocessing
- **Source**: Massachusetts Roads benchmark (1500×1500 RGB orthophotos with binary road masks)  
- **Tiling**: Each image → 3×3 grid of 512×512 patches → ~7240 tiles  
- **Quick-split**: 1 000 tiles for training, 200 for validation (rapid prototyping)  
- **Filtering**: Discard tiles with < 1 % road coverage  
- **Augmentations**: Random flips, rotations, brightness/contrast adjustments, Gaussian blur  

---

## Methodology

### 1. Baseline (Deliverable 3: Deep ResUNet + Boundary-aware Loss)  
- 5-level encoder–decoder built from residual units (3×3 Conv + BN + ReLU with identity skips)  
- Strided convolutions replace pooling for downsampling  
- Loss schedule: epochs 1–10 → Boundary + Weighted BCE + Dice; epochs 11+ → Weighted BCE + Dice  

### 2. Deliverable 4 (CBAM + Light-ASPP)  
- **CBAM** modules at each bottleneck for channel/spatial attention  
- **Light-weight ASPP** block in the bridge for multi-dilation context  
- Loss: Weighted BCE + Focal Dice (with boundary warm-up)  

### 3. Deliverable 5 (Nested U-Net++ + Attention Gates)  
- Dense nested skip-paths fuse high-res encoder features at every depth  
- **Attention Gates** filter skip connections for relevance  
- **Deep supervision** via auxiliary heads at each scale  
- Loss: Focal Tversky (α = 0.7, β = 0.3) with dynamic boundary weight (λ=1 for epochs 1–10)  
- CosineAnnealingLR scheduler for smooth convergence  

### 4. Post-Processing: B-snake Active Contours  
- Initialize cubic B-spline control points every 64 connected pixels  
- Iteratively minimize perpendicular distance to the true road edge  
- Outputs smooth, topologically consistent polylines 

---

## Evaluation & Results

| Model                                                                                                        | IoU   | Precision | Recall | F1    | F<sub>2.0</sub> |
| ------------------------------------------------------------------------------------------------------------ | ----- | --------- | ------ | ----- | --------------- |
| **Baseline (Deep ResUNet + W-BCE + Dice + Boundary)**                                                        | 0.614 | 0.800     | 0.726  | 0.761 | –               |
| **Deliverable 4 (ResUNet + CBAM + Light-ASPP, W-BCE + Focal Dice)**                                  | 0.606 | 0.722     | 0.790  | 0.754 | **0.776**       |
| **Deliverable 5 (NestedUNet_v2 + Attn-Gates + Light-ASPP, W-BCE + Focal Tversky, CosineAnnealing)** | 0.608 | 0.770     | 0.743  | 0.756 | 0.748           |

### 1. Overall Trends
- **Stable segmentation quality**: IoU ≈ 0.61 and F1 ≈ 0.76 across all models.  
- **Deliverable 5** offers the best precision/recall balance, with a slight IoU uptick (0.608 vs. 0.606).

### 2. Deliverable 4 → Deliverable 5 Improvements
- Nested skip-paths and Attention Gates focus features more effectively.  
- Focal Tversky + CosineAnnealingLR smooths convergence and improves precision (+5 pts) with only marginal recall loss (–4.7 pts).  
- Net effect: +0.2 pt F1 and a more robust error trade-off.

### 3. Loss & Scheduling Effects
- Composite multi-term train loss (deep supervision, boundary, Lovász) inflates train/val loss gap.  
- CosineAnnealingLR stabilizes validation loss mid-run, avoiding early plateaus.

---

## Authors & Acknowledgments
**Group 3**  
– Zain Abdullah Ahmad (27100215)  
– Hamid Ali Khan (27100156)

Thanks to the MIT Lincoln Lab team for the Massachusetts Roads dataset and Deep Learning Teaching Staff at LUMS
