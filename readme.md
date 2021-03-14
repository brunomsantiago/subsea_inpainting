# Subsea Inpainting
### Removing overlays from subsea inspection videos <br><br>

* [1. Introduction](#1-introduction)
* [2. The Dataset](#2-the-dataset)
* [3. Inpainting methods tested](#3-inpainting-methods-tested)
  + [3.1. Individual image methods (frame by frame)](#31-individual-image-methods--frame-by-frame-)
    - [3.1.1. OpenCV](#311-opencv)
    - [3.1.2. Deepfill v1](#312-deepfill-v1)
    - [3.1.3. Hifill](#313-hifill)
  + [3.2. Video methods (frame propagation)](#32-video-methods--frame-propagation-)
    - [3.2.1. FGVC](#321-fgvc)
* [4. Testing Code](#4-testing-code)
* [5. Results](#5-results)
  + [5.1. Overview](#51-overview)
  + [5.2. Quality Assessment](#52-quality-assessment)
  + [5.3 Processing times](#53-processing-times)
* [6. Discussion](#6-discussion)
  + [6.1. Quality](#61-quality)
  + [6.2. Processing time](#62-processing-time)
  + [6.3. Robustness](#63-robustness)
  + [6.4 Easy of use](#64-easy-of-use)
- [7. Downloads](#7-downloads)

## 1. Introduction

Oil and Gas industry generates thousands of hours of subsea inspection videos per day. The Oil and Gas operators also have millions of hours of archived video. <br>
These underwater videos are usually acquired by Remote Operated Vehicles (ROVs) and are crucial to the integrity management strategy of subsea assets, like pipelines and Wet Christmas Trees (WCTs), allowing Oil and Gas companies to operate their offshore fields safely. <br>

Most of these videos have an "burned overlay", showing real time metadata like ROV coordinates, water depth and heading. On one hand the burned overlay guarantees the metadata is close to the image data even after manipulations like making short video clips and extracting frames to indiviudal images. On the other hand, the overlay may obstruct important image data from users or confuse image processing algorithms. <br>

<p align="center">
<br>
  <img src="https://github.com/brunomsantiago/subsea_inpainting/raw/main/images/typical_subsea_frame.png">
  <br><b>Figure 1 - Typical Subsea Frame with overlay metadata</b>
</p>

The main goal of this work is an experiment to see if current State of Art inpainting methods are ready to remove overlays from subsea inspection videos. The methods were tested "as is" (without any fine tuning for subsea inspection) and the results were evaluated visually.

As an intermediate step two other contributions were made. A [Subsea Inpainting Dataset](https://www.kaggle.com/brunomsantiago/subsea-inpainting-dataset) and a visualization library named [viajen](https://github.com/brunomsantiago/viajen) (**V**iew **I**mages as **A**nimation in **J**upyter and **E**quivalent **N**otebooks).


## 2. The Dataset

The dataset is composed of 13 small clips of subsea operations that were extracted from 5 videos. It also contains masks covering the overlay for each clip. Each clip has between 20 and 80 frames. The clips are this short because one of the inpainting methods tested, without modifications, is not able to process long sequences, running out of memory.

Both the clips and masks are provided as individual PNG images with the same name in different folders. The masks are black and white images. White means inpaint here and black means keep the original content.

 <p align="center">
 <br>
   <img src="https://github.com/brunomsantiago/subsea_inpainting/raw/main/images/frame_plus_mask.png">
   <br><b>Figure 2 - A frame, its mask and the mask applied on the frame</b>
 </p>

The dataset was created by watching and downloading lots of youtube videos of subsea operations, mostly subsea inspections. Initially 43 videos were selected for further analysis and in the end only 5 was kept. These 5 videos are most similar to private videos I have seen on my work experience at Oil and Gas industry, however not exactly similar.

Several steps were executed to transform the originals videos into the clips. First I extracted frames to individual images. Then, using a image viewer, I chose which frames to keep and which to delete. Next I created a mask template using a image editing software. Finally I made several copies of this mask template matching the individual frames filenames. The code that supported some of the steps is available in [creating_the_dataset.ipynb](https://nbviewer.jupyter.org/github/brunomsantiago/subsea_inpainting/blob/main/other_notebooks/creating_the_dataset.ipynb).

For download and more details about the folder structure of the dataset see [Subsea Inpainting Dataset at Kaggle Datasets](https://www.kaggle.com/brunomsantiago/subsea-inpainting-dataset). As alternative mirror you can download a [single zip](https://drive.google.com/file/d/1OaTLKxkgKlAXMD4PFeHu4YxVVR_nqrkL/view?usp=sharing) (334 MB) from google drive, which is specifically useful if you need to copy the dataset to your own google drive and run the colab notebooks in section 4.

## 3. Inpainting methods tested

### 3.1. Individual image methods (frame by frame)

These methods are designed to used on individual images. When used on videos they are applied frames by frame, without any data propagation from nearby frames, which may lead to inconsistencies between them.

#### 3.1.1. OpenCV

The first method tested was OpenCV Navier-Stokes Image Inpainting (see [OpenCV inpainting tutorial](https://docs.opencv.org/master/df/d3d/tutorial_py_inpainting.html)) which is based on a [paper from 2001](https://ieeexplore.ieee.org/document/990497). It is a classic computer vision method, which have better results removing small defects, thin defects or larger defects on regular backgrounds.

I didn't have great expectations about this method for removing overlay from subsea videos, but included it in the analysis as baseline. This baseline is useful both for inpainting results and processing time. More advanced methods must have inpainting results that look much better.  methods ready for production should look for processing times as close a possible as opencv method.

#### 3.1.2. Deepfill v1

Deepfill v1 ([paper](https://arxiv.org/abs/1801.07892), [code](https://github.com/JiahuiYu/generative_inpainting/tree/v1.0.0), [video](https://youtu.be/xz1ZvcdhgQ0)) and v2 ([paper](https://arxiv.org/abs/1806.03589), [code](https://github.com/JiahuiYu/generative_inpainting/tree/v2.0.0), [video](https://youtu.be/uZkEi9Y2dj4)) are image inpainting methods based on deep neural networks. From Deepfill v1 paper abstract:

> "... a new deep generative model-based approach which can not only synthesize novel image structures but also explicitly utilize surrounding image features as references during network training to make better predictions. The model is a feed-forward, fully convolutional neural network which can process images with multiple holes at arbitrary locations and with variable sizes during the test time"

Although Deepfill v1 and v2 have their official implementations on github, I didn't used them. I used v1 implementation from FGVC (see more details below), which uses Deepfill v1 as part of its processing pipeline.

#### 3.1.3. Hifill

Hifill ([paper](https://arxiv.org/abs/2005.09704), [code](https://github.com/Atlas200dk/sample-imageinpainting-HiFill), [video](https://youtu.be/Q7mX5Bstv7U)) is a image inpainting method similar to Deepfill, which is designed to work better on high resolution images, with larger roles and easier to adapt for new contexts (with few images). From the abstract:

> ... a Contextual Residual Aggregation (CRA) mechanism that can produce high-frequency residuals for missing contents by weighted aggregating residuals from contextual patches, thus only requiring a low-resolution prediction from the network. Since convolutional layers of the neural network only need to operate on low-resolution inputs and outputs, the cost of memory and computing power is thus well suppressed. Moreover, the need for high-resolution training datasets is alleviated. In our experiments, we train the proposed model on small images with resolutions 512x512 and perform inference on high-resolution images, achieving compelling inpainting quality. Our model can inpaint images as large as 8K with considerable hole sizes, which is intractable with previous learning-based approaches.

### 3.2. Video methods (frame propagation)

Video inpainting methods differ from image methods in two points. First it uses data from nearby frames to complete the current frame, as the camera or object movement may cause the data missing from one frame to be available in the others. Second it actively try to achieve temporal consistency, inpainting nearby frames in a similar manner.

#### 3.2.1. FGVC

FGVC ([paper](https://arxiv.org/abs/2009.01835), [code](https://github.com/vt-vl-lab/FGVC), [video](https://youtu.be/CHHVPxHT7rc)) is a video inpainting method with a interest pipeline. First it estimates flow, complete its edges to create a smoothed completed flow. Then it uses this new flow to guide a search for missing pixels from the current frame in other frames. Next it completes still missing pixels using a image inpainting method. Finally it blends all frames together in a seamless way. In addition this whole process is done forward and backward, and is also iterative.

FGVC uses 3 neural networks. One to estimate optical flow (originally FlowNet2, currently RAFT), other to complete edges on the optical flow, and another to complete missing pixels (Deepfill v1). Even though the GPU intensive neural networks are important parts of the pipeline, most of processing time seems to be used looking for missing pixels and blending frames, which are CPU intensive operations.

## 4. Testing Code

| Inpainting Method | Kaggle Notebook                    | Colab Notebook |
|-------------------|------------------------------------|----------------|
| OpenCV            | [view in nbviewer](https://nbviewer.jupyter.org/github/brunomsantiago/subsea_inpainting/blob/main/notebooks/kaggle/subsea-inpainting-01-opencv.ipynb) \| [play on kaggle](https://www.kaggle.com/brunomsantiago/subsea-inpainting-01-opencv) | [view in github](https://github.com/brunomsantiago/subsea_inpainting/blob/main/notebooks/colab/subsea-inpainting-01-opencv.ipynb) \| [play on colab](https://colab.research.google.com/github/brunomsantiago/subsea_inpainting/blob/main/notebooks/colab/subsea-inpainting-01-opencv.ipynb)          |
| Deepfill v1       | [view in nbviewer](https://nbviewer.jupyter.org/github/brunomsantiago/subsea_inpainting/blob/main/notebooks/kaggle/subsea-inpainting-02-deepfill-v1.ipynb) \| [play on kaggle](https://www.kaggle.com/brunomsantiago/subsea-inpainting-02-deepfill-v1) | TO DO          |
| Hifill            | [view in nbviewer](https://nbviewer.jupyter.org/github/brunomsantiago/subsea_inpainting/blob/main/notebooks/kaggle/subsea-inpainting-03-hifill.ipynb) \| [play on kaggle](https://www.kaggle.com/brunomsantiago/subsea-inpainting-03-hifill) | TO DO          |
| FGVC              | [view in nbviewer](https://nbviewer.jupyter.org/github/brunomsantiago/subsea_inpainting/blob/main/notebooks/kaggle/subsea-inpainting-04-fgvc.ipynb) \| [play on kaggle](https://www.kaggle.com/brunomsantiago/subsea-inpainting-04-fgvc) | TO TO          |

## 5. Results

### 5.1. Overview

The output of the inpainting are the inpainted versions of individual frames images (PNG files). FGVC also outputs the optical flow, the original and the completed one. All generated images, from the 13 clips, can be download as [single zip](https://drive.google.com/file/d/1jUv7pgKGsEN3_L5F_qsxe-QKISmY51p7/view?usp=sharing) (775 MB) from google drive.

For better visualization I made the results into Animated GIFs. There are for each subsea inpainting dataset clip:
- A GIF comparing the 4 methods in a 4x4 grid
  <p align="center">
    <img src="https://github.com/brunomsantiago/subsea_inpainting/raw/main/images/01a_05_full_compare.gif">
    <br><b>Figure 3 - 4x4 inpainting comparison</b><br> (top left: Opencv | top right: Deepfill v1 | bottom left: Hifill | bottom right: FGVC)
  </p>  

- Several GIFs comparing each method to original masked frames
  <p align="center">
    <img src="https://github.com/brunomsantiago/subsea_inpainting/raw/main/images/01a_04_fgvc_compare.gif">
    <br><b>Figure 4 - Side by side comparison</b><br> (left: Original masked clip | right: Inpainted by FGVC)
  </p>  

- A GIF to analyse FGVC optical flow
  <p align="center">
    <img src="https://github.com/brunomsantiago/subsea_inpainting/raw/main/images/01a_06_fgvc_compare_flows.gif">
    <br><b>Figure 5 - FGVC Optical Forward Flow</b> <br> top left: Original masked clip | top right: Inpainted by FGVC | bottom left: Original Optical Flow | bottom right: Completed Optical Flow)
  </p>    

All the GIFs are available for download as [single zip](https://drive.google.com/file/d/1dmw2F-WMNkEnjuYB3gDUyuYCqq5v1z_M/view?usp=sharing) (1.4 GB) from google drive. The 4x4 grids comparing the methods are also available on imgmur and linked on the table below. On imgmur I recommend view in full screen for better visualization.
<br>

### 5.2. Quality Assessment

The results were evaluated visually and subjectively classified in a four grade rank:
 - **Excellent** - It is not easy to detect the images were inpainted. It almost seems like magic.
 - **Good** - At least in some regions is easy to detect it was inpainted. Despite of detection it doesn't look too bad and it maybe not so easy to detect looking into individual frames.
 - **Fair** - It easy to detect the images, even on individual frames. However the inpainting artifacts aren't outstanding.
 - **Poor** - It is very easy to detect the image was inpainted, usually because of weird outstanding inpainting artifacts.


| Clip                                                             | Number of Frames | FGVC         | Hifill       | Deepfill | Opencv |
| :-:                                                              | :--:             | :-:          | :-:          | :-:      | :-:    |
| **01a** <sub>([4x4 grid](https://imgur.com/a/v2mNWF8/all))</sub> | 20               | Excellent    | Fair         | Poor     | Poor   |
| **01b** <sub>([4x4 grid](https://imgur.com/a/1jsc1ph/all))</sub> | 50               | Excellent    | Fair         | Poor     | Poor   |
| **01c** <sub>([4x4 grid](https://imgur.com/a/hwFFKAL/all))</sub> | 40               | Excellent    | Good         | Poor     | Poor   |
| **02a** <sub>([4x4 grid](https://imgur.com/a/EfOiKEx/all))</sub> | 40               | Fair         | Poor         | Good     | Poor   |
| **03a** <sub>([4x4 grid](https://imgur.com/a/ZioikGt/all))</sub> | 40               | Poor -> Good | Good -> Fair | Poor     | Poor   |
| **03b** <sub>([4x4 grid](https://imgur.com/a/SQGQl4r/all))</sub> | 20               | Good         | Poor         | Fair     | Poor   |
| **04a** <sub>([4x4 grid](https://imgur.com/a/qwISx6l/all))</sub> | 30               | Excellent    | Fair         | Good     | Poor   |
| **05a** <sub>([4x4 grid](https://imgur.com/a/57B1Gex/all))</sub> | 69               | Poor         | Poor         | Good     | Fair   |
| **05b** <sub>([4x4 grid](https://imgur.com/a/S68m5k9/all))</sub> | 40               | Poor         | Poor         | Good     | Fair   |
| **05c** <sub>([4x4 grid](https://imgur.com/a/rP11OD1/all))</sub> | 80               | Poor         | Poor         | Poor     | Fair   |
| **05d** <sub>([4x4 grid](https://imgur.com/a/4LhsFAa/all))</sub> | 30               | Poor         | Fair         | Fair     | Poor   |
| **05e** <sub>([4x4 grid](https://imgur.com/a/ARnLu9G/all))</sub> | 30               | Poor -> Good | Poor         | Fair     | Poor   |
| **05f** <sub>([4x4 grid](https://imgur.com/a/9orp3cC/all))</sub> | 40               | Fair         | Fair         | Fair     | Poor   |

### 5.3 Processing times

Opencv processing time used as baseline (1x)

| Clip     | Number of Frames | FGVC                                            | Hifill                                        | Deepfill                                    | Opencv                                      |
| :-:      | :-:              | :-:                                             | :-:                                           | :-:                                         | :-:                                         |
| **01a**  | 20               | 229 s <br> <sub> (11.45 s/frame) (72 x) </sub>  | 48 s <br> <sub> (2.40 s/frame) (15 x) </sub>  | 8 s <br> <sub> (0.42 s/frame) (3 x) </sub>  | 3 s <br> <sub> (0.16 s/frame) (1 x) </sub>  |
| **01b**  | 50               | 576 s <br> <sub> (11.53 s/frame) (56 x) </sub>  | 134 s <br> <sub> (2.68 s/frame) (13 x) </sub> | 26 s <br> <sub> (0.53 s/frame) (3 x) </sub> | 10 s <br> <sub> (0.21 s/frame) (1 x) </sub> |
| **01c**  | 40               | 512 s <br> <sub> (12.80 s/frame) (64 x) </sub>  | 96 s <br> <sub> (2.39 s/frame) (12 x) </sub>  | 16 s <br> <sub> (0.39 s/frame) (2 x) </sub> | 8 s <br> <sub> (0.20 s/frame) (1 x) </sub>  |
| **02a**  | 40               | 622 s <br> <sub> (15.54 s/frame) (66 x) </sub>  | 118 s <br> <sub> (2.95 s/frame) (13 x) </sub> | 36 s <br> <sub> (0.91 s/frame) (4 x) </sub> | 9 s <br> <sub> (0.24 s/frame) (1 x) </sub>  |
| **03a**  | 40               | 649 s <br> <sub> (16.23 s/frame) (85 x) </sub>  | 95 s <br> <sub> (2.38 s/frame) (13 x) </sub>  | 11 s <br> <sub> (0.29 s/frame) (2 x) </sub> | 8 s <br> <sub> (0.19 s/frame) (1 x) </sub>  |
| **03b**  | 20               | 286 s <br> <sub> (14.29 s/frame) (64 x) </sub>  | 48 s <br> <sub> (2.38 s/frame) (11 x) </sub>  | 6 s <br> <sub> (0.32 s/frame) (1 x) </sub>  | 4 s <br> <sub> (0.23 s/frame) (1 x) </sub>  |
| **04a**  | 30               | 411 s <br> <sub> (13.69 s/frame) (75 x) </sub>  | 72 s <br> <sub> (2.39 s/frame) (13 x) </sub>  | 10 s <br> <sub> (0.33 s/frame) (2 x) </sub> | 6 s <br> <sub> (0.18 s/frame) (1 x) </sub>  |
| **05a**  | 69               | 997 s <br> <sub> (14.46 s/frame) (46 x) </sub>  | 174 s <br> <sub> (2.52 s/frame) (8 x) </sub>  | 40 s <br> <sub> (0.58 s/frame) (2 x) </sub> | 22 s <br> <sub> (0.31 s/frame) (1 x) </sub> |
| **05b**  | 40               | 489 s <br> <sub> (12.23 s/frame) (71 x) </sub>  | 98 s <br> <sub> (2.45 s/frame) (14 x) </sub>  | 12 s <br> <sub> (0.30 s/frame) (2 x) </sub> | 7 s <br> <sub> (0.17 s/frame) (1 x) </sub>  |
| **05c**  | 80               | 1104 s <br> <sub> (13.80 s/frame) (32 x) </sub> | 194 s <br> <sub> (2.42 s/frame) (6 x) </sub>  | 24 s <br> <sub> (0.30 s/frame) (1 x) </sub> | 34 s <br> <sub> (0.43 s/frame) (1 x) </sub> |
| **05d**  | 30               | 345 s <br> <sub> (11.50 s/frame) (64 x) </sub>  | 72 s <br> <sub> (2.38 s/frame) (13 x) </sub>  | 9 s <br> <sub> (0.31 s/frame) (2 x) </sub>  | 5 s <br> <sub> (0.18 s/frame) (1 x) </sub>  |
| **05e**  | 30               | 343 s <br> <sub> (11.44 s/frame) (70 x) </sub>  | 73 s <br> <sub> (2.43 s/frame) (15 x) </sub>  | 9 s <br> <sub> (0.29 s/frame) (2 x) </sub>  | 5 s <br> <sub> (0.16 s/frame) (1 x) </sub>  |
| **05f**  | 40               | 452 s <br> <sub> (11.31 s/frame) (72 x) </sub>  | 94 s <br> <sub> (2.36 s/frame) (15 x) </sub>  | 11 s <br> <sub> (0.28 s/frame) (2 x) </sub> | 6 s <br> <sub> (0.16 s/frame) (1 x) </sub>  |

## 6. Discussion

### 6.1. Quality

- Most methods perform badly on low brightness areas
  - However opencv tends to perform better in these areas
  - FGVC is the worse in these areas. It seems to have issues tracking pixels between frames in low brightness area.

### 6.2. Processing time

### 6.3. Robustness

### 6.4 Easy of use

# 7. Downloads
