# Day 8 — Dataset / Annotation Quality Report

## Project

**Project:** AI-Enabled Underwater Garbage Detection System  
**Dataset:** TrashCAN 1.0  
**Task:** Dataset / Annotation Quality Audit

---

## 1. Purpose of the Audit

The main purpose of this audit was to understand the dataset before using it for further model development. I focused on checking whether the image files and their annotations are properly organized and paired, whether the images can be read without errors, what resolutions are present, and what the annotation format looks like. The goal at this stage was not to modify the original dataset, but to identify possible issues that should be addressed before preprocessing and model training.

---

## 2. Dataset Structure

The provided original dataset contains:
```text
original_data/
├── images/
├── annotations/
└── README.txt
```
The `images` directory contains individual `.jpg` image files.
The `annotations` directory contains individual `.json` annotation files generated using Supervisely.
The image and annotation filenames follow this pattern:

```text
Image:
vid_000002_frame0000013.jpg
Annotation:
vid_000002_frame0000013.jpg.json
```
Because of the `.jpg.json` naming convention, I matched the annotation filenames with the image filenames by removing the `.jpg.json` suffix before comparing them.

---

## 3. Dataset Inventory

The initial count gave the following results:
| Item | Count |
|---|---:|
| Total images | 7,214 |
| Total annotation files | 7,212 |
| Images without annotations | 2 |
| Annotations without corresponding images | 0 |

Most of the images therefore have a corresponding annotation file. There are only two images for which an annotation file could not be found.
The two images are:
```text
vid_000270_frame0000041.jpg
vid_000270_frame0000044.jpg
```
No annotation files were found that were completely unmatched with an image.

---

## 4. Image–Annotation Pairing

I checked the image and annotation filenames programmatically instead of assuming that all files were correctly paired.
The result was:
```text
Total images: 7214
Total annotations: 7212

Images without annotations: 2
Annotations without images: 0
```
The missing annotation issue is limited to two frames from `vid_000270`. I would not directly delete these two images. They should first be inspected to understand whether they contain objects that should have been annotated or whether they were intentionally left without annotations.

### Proposed action
- Inspect frame 41 and frame 44 manually.
- If they contain relevant trash objects and should be part of the labelled dataset, create or recover the missing annotations.
- If they are intentionally unannotated or unsuitable for the final training set, exclude them from the final prepared dataset and document the reason.

---

## 5. Image Integrity Check

I checked all `.jpg` images using Python and Pillow to see whether they could be opened and verified successfully.
The result was:
```text
Corrupt images: 0
```
No corrupt or unreadable JPG files were detected during this check.
This is a positive result because the image files themselves appear to be readable.
However, being readable does not necessarily mean that an image is visually suitable for training. Visual issues such as blur, poor visibility, turbidity, low contrast, occlusion, or very small objects still need to be checked separately.

---

## 6. Image Resolution Analysis

The dataset contains two different image resolutions.
| Resolution | Number of images |
|---|---:|
| 480 × 270 | 3,967 |
| 480 × 360 | 3,247 |
| **Total** | **7,214** |

The dataset therefore contains images with two different aspect ratios:
- **480 × 270** → 16:9
- **480 × 360** → 4:3
Directly resizing every image to one fixed width and height without considering the original aspect ratio could distort objects in the images.

### Proposed action
During preprocessing, I plan to use a consistent approach that avoids unnecessary distortion, such as aspect-ratio-preserving resizing with padding/letterboxing, depending on the final model and input requirements.

---

## 7. Annotation Format

I inspected an actual annotation JSON file to understand how the labels are represented.
The annotation follows the Supervisely JSON structure. It contains information such as:
```text
description
tags
size
objects
```

Each object can contain information such as:
```text
id
classId
geometryType
classTitle
tags
bitmap
```

The sample annotation that I inspected had:
```text
Image width: 480
Image height: 270
Class: trash
Geometry type: bitmap
```

The object also contained additional attributes such as:

```text
material
crushed/broken
decay
instance
```

An important observation is that the annotation uses **bitmap geometry**. Therefore, the original annotation is not simply a bounding box stored as four coordinates. This is important for the later modelling stage because the annotation format may need to be converted depending on whether the final project uses object detection, segmentation, or another vision approach. For Day 8, I have kept the original annotation format unchanged and focused on understanding and auditing it.

---

## 8. Annotation Quality Observations

From the sample annotation inspected, the annotation contains:

- a defined image size,
- an object ID,
- a class ID,
- a class name,
- a geometry type,
- object-level tags,
- and bitmap information.

The sample annotation was structurally meaningful and contained more information than just a class label.

A full annotation-quality conclusion, however, should not be based on only one sample. The remaining annotation checks should be applied across all annotation files before the dataset is finalized.

---

## 9. Findings

### Positive findings
1. The dataset has a clear separation between images and annotations.
2. There are **7,214 JPG images** available.
3. There are **7,212 annotation JSON files**.
4. No orphan annotation files were found.
5. No corrupt JPG images were detected.
6. The annotation format could be identified as Supervisely JSON.
7. The inspected annotation uses bitmap geometry and contains object-level information.

### Issues / points requiring attention
1. **Two images do not have corresponding annotations:**
   - `vid_000270_frame0000041.jpg`
   - `vid_000270_frame0000044.jpg`
2. **Two image resolutions are present:**
   - 480 × 270
   - 480 × 360
3. **The original annotations use bitmap geometry**, so the annotation format needs to be considered carefully when selecting the final model and preparing the data.

---

## 10. Correction Plan

| Issue | Proposed action | Priority |
|---|---|---|
| Missing annotation for frame 41 | Inspect the image and recover/create an annotation if required | High |
| Missing annotation for frame 44 | Inspect the image and recover/create an annotation if required | High |
| Different image resolutions | Use a consistent aspect-ratio-safe preprocessing method | Medium |
| Supervisely bitmap annotations | Keep the original annotations unchanged and convert them only after the final model format is decided | Medium |
| Possible visual-quality problems | Inspect representative images for blur, turbidity, visibility, occlusion and small objects | High |
| Dataset split | Check source/video-level grouping before creating train/validation/test splits | High |
| Duplicate samples | Run duplicate analysis before finalizing the dataset | High |

---

## 12. Conclusion

From the checks completed so far, the dataset looks mostly well organized at the file level. The main issue identified at this stage is the absence of annotations for two frames from `vid_000270`. The images themselves are readable, and no corrupt JPG files were found. Another important observation is that the dataset contains two different image resolutions and uses Supervisely bitmap annotations. These points should be taken into account before preprocessing and model training. The next step is to complete the remaining duplicate, annotation-distribution, visual-quality, and split-integrity checks. Based on those results, the final preprocessing and annotation correction plan can be finalized.

---

## Dataset Reference

The dataset used for this audit is **TrashCAN 1.0**.
The associated paper is:
**Jungseok Hong, Michael Fulton, and Junaed Sattar.  
“TrashCAN: A Semantically Segmented Dataset towards Visual Detection of Marine Debris.”**
Paper: https://arxiv.org/abs/2007.08097
The paper should be cited when using the TrashCAN dataset.

---
