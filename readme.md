# Subsea Inpainting
### Removing overlays from subsea inspection videos <br><br>

----

This work is my final assignment to finish the course Business Inteligence Master at PUC University, Rio de Janeiro, Brasil. For more details see [abstract (in Portuguese)](https://github.com/brunomsantiago/subsea_inpainting/blob/main/monografia.pt-BR.md).

---
### Table of Contents

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
  + [6.5 Final remarks](#65-final-remarks)
- [7. Downloads](#7-downloads)

---

## 1. Introduction

Oil and Gas industry generates thousands of hours of subsea inspection videos per day. The Oil and Gas operators also have millions of hours of archived video. These underwater videos are usually acquired by Remote Operated Vehicles (ROVs) and are crucial to the integrity management strategy of subsea assets, like pipelines and Wet Christmas Trees (WCTs), allowing Oil and Gas companies to operate their offshore fields safely. There are lot of potential value in secondary uses of these videos, which can ony be tapped by applying computer vision and artificial intelligence techniques.

Most of these videos have an "burned overlay", showing metadata like date, hour and ROV coordinates. On one hand the burned overlay guarantees the metadata is close to the image data even after manipulations like making short video clips and extracting frames to indiviudal images. On the other hand, the overlay may obstruct important image data from users or confuse image processing algorithms.

<p align="center">
<br>
  <img src="https://github.com/brunomsantiago/subsea_inpainting/raw/main/images/typical_subsea_frame.png">
  <br><b>Figure 1 - Typical Subsea Frame with overlay metadata</b>
</p>

The main goal of this work is to see if current State of Art inpainting methods are ready to remove overlays from subsea inspection videos. Four methods were evaluated, three of them based on deep neural networks. As an intermediate step two other contributions were made. A [Subsea Inpainting Dataset](https://www.kaggle.com/brunomsantiago/subsea-inpainting-dataset) and a visualization library named [viajen](https://github.com/brunomsantiago/viajen) (**V**iew **I**mages as **A**nimation in **J**upyter and **E**quivalent **N**otebooks).


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

The code used to inpaint the clips are available in folder [notebooks_inpainting](https://github.com/brunomsantiago/subsea_inpainting/tree/main/notebooks_inpainting) of this repository. There are two notebooks for each evaluated method (OpenCV, Deepfill V1, Hifill, FGVC), ready do play in Kaggle, other ready to play on Google Colab. Those platforms are great and have free GPU notebooks, however there are trade-offs and decided to provide code for both of them.

In my experience Google Colab have faster GPU instances, more GPU time in the free tier and easier integration with google drive, which make it more suitable for longer running notebooks and to retrieve results for further analysis. However the initial dataset setup is a little harder.

On the other hand, Kaggle already have the dataset ready to use, making it easier to start playing with the data. Kaggle free GPU instances, however, need the users to register themselves and confirming their profiles with a valid cellphone. For people who never used Kaggle or don't intend to use, Colab maybe a better option.

All these notebooks can be viewed here on github (which sometimes is slow to display notebooks) and on nbviewer (which sometimes gives 404 errors when trying to display from repositories with many notebooks like this one, specially recently updated notebooks). If you want to just preview the code, I suggest trying first the nbviewer version.

The notebooks need the models (codes and pre-trained weights) of Deepfill, Hifill and FGVC to work properly. All the models are available on the [models branch](https://github.com/brunomsantiago/subsea_inpainting/tree/models) of this repository, which are automatically cloned by the notebooks.

| Inpainting Method | Kaggle Notebook                    | Colab Notebook |
|-------------------|------------------------------------|----------------|
| OpenCV | [github](https://github.com/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/kaggle/kaggle-subsea-inpainting-01-opencv.ipynb) \| [nbviewer](https://nbviewer.jupyter.org/github/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/kaggle/kaggle-subsea-inpainting-01-opencv.ipynb) \| [play on kaggle](https://www.kaggle.com/brunomsantiago/subsea-inpainting-01-opencv) | [github](https://github.com/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/colab/colab-subsea-inpainting-01-opencv.ipynb) \| [nbviewer](https://nbviewer.jupyter.org/github/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/colab/colab-subsea-inpainting-01-opencv.ipynb) \| [play on colab](https://colab.research.google.com/github/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/colab/colab-subsea-inpainting-01-opencv.ipynb) |
| Deepfill v1 | [github](https://github.com/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/kaggle/kaggle-subsea-inpainting-02-deepfill-v1.ipynb) \| [nbviewer](https://nbviewer.jupyter.org/github/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/kaggle/kaggle-subsea-inpainting-02-deepfill-v1.ipynb) \| [play on kaggle](https://www.kaggle.com/brunomsantiago/subsea-inpainting-02-deepfill-v1) | [github](https://github.com/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/colab/colab-subsea-inpainting-02-deepfill.ipynb) \| [nbviewer](https://nbviewer.jupyter.org/github/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/colab/colab-subsea-inpainting-02-deepfill.ipynb) \| [play on colab](https://colab.research.google.com/github/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/colab/colab-subsea-inpainting-02-deepfill.ipynb) |
| Hifill | [github](https://github.com/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/kaggle/kaggle-subsea-inpainting-03-hifill.ipynb) \| [nbviewer](https://nbviewer.jupyter.org/github/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/kaggle/kaggle-subsea-inpainting-03-hifill.ipynb) \| [play on kaggle](https://www.kaggle.com/brunomsantiago/subsea-inpainting-03-hifill) | [github](https://github.com/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/colab/colab-subsea-inpainting-03-hifill.ipynb) \| [nbviewer](https://nbviewer.jupyter.org/github/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/colab/colab-subsea-inpainting-03-hifill.ipynb) \| [play on colab](https://colab.research.google.com/github/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/colab/colab-subsea-inpainting-03-hifill.ipynb) |
| FGVC | [github](https://github.com/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/kaggle/kaggle-subsea-inpainting-04-fgvc.ipynb) \| [nbviewer](https://nbviewer.jupyter.org/github/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/kaggle/kaggle-subsea-inpainting-04-fgvc.ipynb) \| [play on kaggle](https://www.kaggle.com/brunomsantiago/subsea-inpainting-04-fgvc) | [github](https://github.com/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/colab/colab-subsea-inpainting-04-fgvc.ipynb) \| [nbviewer](https://nbviewer.jupyter.org/github/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/colab/colab-subsea-inpainting-04-fgvc.ipynb) \| [play on colab](https://colab.research.google.com/github/brunomsantiago/subsea_inpainting/blob/main/notebooks_inpainting/colab/colab-subsea-inpainting-04-fgvc.ipynb) |

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

 - The methods were executed on Google Colab free GPU instances in march 2021, which means the GPU was probably a K80.
 - Opencv processing times was used as baseline (1x).

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

FGVC is an truly awesome work and sometimes its results are amazing, looking like magic. Unfortunately sometimes the results are not so good. Deepfill and Hifill results were not even close to the quality of FGVC.

I would need to make additional clips to confirm, but it seems to me FGVC has quality issues with longer sequences of frames. It could be further investigated by making longer clips of the good results and smaller versions of the bad ones.

All three learning based methods (FGVC, Deepfill, Hifill), but mostly FGVC, seems to have issue with some semi-uniform backgrounds, specially with low brightness and high noise. This issues appears on clips 3a, 5a, 5b, 5c, 5d and 5e.
 - I would need to investigate further, but it seems to me FGVC has false positive matching of pixels between frames in such scenario, propagating wrong data. Deepfill, which is used by FGVC to hallucinate completely missing pixels performed much better in these clips, except on clip 3a.
 - The low quality of FGVC results on these clips doesn't seem to be associated with the optical flow. The completed flow seems good.
 - Just for the records, Opencv method has good results in these areas and seemingly only in these areas.

**On the quality aspect none of the methods tested is production ready yet**, at least not for this application. However all three learning based methods could probably improve a lot by training on a large dataset of subsea inspection images without overlay.

It does not seem too hard to build such dataset, but it's necessary access an Oil and Gas company video archive. In these video the overlay are usually on the top and bottom sections of the frames, and can be easily cropped out. After cropping the aspect ratio would be weird, but this can be fixed with further cropping. This final may generate a single image (with a center crop, for example) or multiple images (for example: "left and right crops" or "left, center and right crops"), depending on the training strategy. The only data annotation needed would be two vertical coordinates to crop out the overlays.


### 6.2. Processing time

The code was executed in Google Colab with free GPU instances, so there is room for improvement with better hardware.

Overall the FGVC processing times were very long, between 11 and 16 seconds per frame. It took as long as 18,4 minutes for a 80 frames long clip. FGVC were between 46x and 72x slower than Opencv.

 It was expected that FGVC times would be slower, as it has a complex pipeline and is a quite new method, product of a research project published just few months ago. On the other hand FGVC seems to have much room for improvement.

 For example, the [FGVC tool code]() seem to compare each frame with all other frames in a sequence, in order to propagate data between them. Probably it is possible to improve its speed without loosing quality focusing the frame propagation to nearby frames.

 Another example of optimization, useful for all methods but specially for FGVC (which is currently CPU bound) is to split long sequences of frames into small clips, maybe with a little overlaping between them. By processing these clips in parallel, with different cores from CPU or GPU, it would be possible to drop the average processing speed to acceptable levels.

  **Without at least a order of magnitude improvement in processing time, FGVC isn't suitable to remove overlay of subsea inspection videos**, which can be very long. A single operation may have many hours of video data.

### 6.3. Robustness

FGVC didn't work with some frame resolutions and aspect ratio, raising errors and stopping the processing. For example, clip 02a original video resolution was 1920x1080. I tried resizing it to a lower resolution, keeping the aspect ratio, but it didn't work either. I had to change the aspect ratio, which I did by cropping content from the sides. FGVC was also not able to process longer sequences of frames, running out of memory. I also tried to process lower resolution videos, to see if would be possible to test longer sequences of frames, and it also didn't work. A production ready method must be more flexible regarding the input.

### 6.4 Easy of use

I admire researchers who publish the code and weights with their papers, making easier for others to test their methods with their own data. However python scripts and jupyter notebooks are not a production ready solution. A good solution for common users should, at least:
 - Have a good graphic interface, desktop or online.
 - Accept video files as inputs
 - Allow the user to draw their own mask on top of the video.
 - Allow process video in batches.

### 6.5 Final remarks

From the four methods tests, overall, the FGVC had the best results. It has potential to evolve into a production ready method, but it is not ready yet, neither in quality nor processing speed. It's worth noting that FGVC was the only video method tested. There are others like Deep Video Inpainting ([paper](https://arxiv.org/abs/1905.02949), [code](https://github.com/mcahny/Deep-Video-Inpainting), [video](https://youtu.be/RtThGNTvkjY)) and Deep Flow-Guided Video Inpainting ([paper](https://arxiv.org/abs/1905.02884), [code](https://github.com/nbei/Deep-Flow-Guided-Video-Inpainting), [video](https://youtu.be/LIJPUsrwx5E)).

I am not sure it is possible to get excellent quality and temporal consistency by using a frame by frame method, like Deepfill and Hifill. However their results seems to have much room for improvement too.

Regardless of the method, broad use of overlay removal from subsea inspection videos will only be achieved with a user friendly software.

# 7. Downloads

The links for relevant steps of this work are available at relevant sections of the text, however here is a recapitulation.
 - **viajen visualiation library** - [Github](https://github.com/brunomsantiago/viajen)
 - **Subsea Inpainting Dataset** - [Kaggle](), [Google Drive](https://drive.google.com/file/d/1OaTLKxkgKlAXMD4PFeHu4YxVVR_nqrkL/view?usp=sharing) (334 MB)
 - **Subsea Inpainting Results - Static Images** - [Google Drive](https://drive.google.com/file/d/1jUv7pgKGsEN3_L5F_qsxe-QKISmY51p7/view?usp=sharing) (775 MB)
 - **Subsea Inpainting Results - Animated GIFs** - [Google Drive](https://drive.google.com/file/d/1dmw2F-WMNkEnjuYB3gDUyuYCqq5v1z_M/view?usp=sharing) (1.4 GB)
