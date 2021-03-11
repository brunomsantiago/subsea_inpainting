# Subsea Inpainting
### Removing overlays from subsea inspection videos <br><br>


## 1. Introduction

Oil and Gas industry generates thousands of hours of subsea inspection videos per day. The Oil and Gas operators also have millions of hours of archived video. <br>
These underwater videos are usually acquired by Remote Operated Vehicles (ROVs) and are crucial to the integrity management strategy of subsea assets, like pipelines and Wet Christmas Trees (WCTs), allowing Oil and Gas companies to operate their offshore fields safely. <br>

Most of these videos have an "burned overlay", showing real time metadata like ROV coordinates, water depth and heading. On one hand the burned overlay guarantees the metadata is close to the image data even after manipulations like making short video clips and extracting frames to indiviudal images. On the other hand, the overlay may obstruct important image data from users or confuse image processing algorithms. <br>

<p align="center">
<br>
  <img src="https://github.com/brunomsantiago/subsea_inpainting/raw/main/images/typical_subsea_frame.png">
  <br><b>Figure 1 - Typical Subsea Frame with overlay metadata</b>
</p>

The main goal of this work is an experiment to see if current State of Art inpainting techniques are ready to remove overlays from subsea inspection videos. The techniques were tested "as is" (without any fine tuning for subsea inspection) and the results were evaluated visually.

As an intermediate step two other contributions were made. A [Subsea Inpainting Dataset](https://www.kaggle.com/brunomsantiago/subsea-inpainting-dataset) and a visualization library named [viajen](https://github.com/brunomsantiago/viajen) (**V**iew **I**mages as **A**nimation in **J**upyter and **E**quivalent **N**otebooks).


## 2. The Dataset

The dataset is composed of 13 small clips of subsea operations that were extracted from 5 videos. It also contains masks covering the overlay for each clip.

Each clip has between 15 and 80 frames. The clips are this short because one of the inpainting techniques tested, without modifications, is not able to process long sequences, running out of memory.

Both the clips and masks are provided as individual PNG images with the same name in different folders.

 The masks are black and white PNG images. White means inpaint here and black means keep the original content.

 <p align="center">
 <br>
   <img src="https://github.com/brunomsantiago/subsea_inpainting/raw/main/images/frame_plus_mask.png">
   <br><b>Figure 2 - A frame, its mask and the mask applied on the frame</b>
 </p>

The dataset was created by watching and downloading lots of youtube videos of subsea operations, mostly subsea inspections. Initially 43 videos were selected for further analysis and in the end only 5 was kept. These 5 videos are most similar to private videos I have seen on my work experience at Oil and Gas industry, however not exactly similar.

For download and more details about the folder structure of the dataset see [Subsea Inpainting Dataset at Kaggle Datasets](https://www.kaggle.com/brunomsantiago/subsea-inpainting-dataset).

## 3. Inpainting techniques tested

#### 3.1 OpenCV

The first technique tested was OpenCV Navier-Stokes Image Inpainting (see [OpenCV tutorial](https://docs.opencv.org/master/df/d3d/tutorial_py_inpainting.html)) which is based on a [paper from 2001](https://ieeexplore.ieee.org/document/990497).
It is a classic computer vision method, applied to individually to frames.

I didn't have great expectations about this technique, but included it in the analysis as baseline. Both in quality and processing time.

#### 3.2. Deepfill v1

#### 3.3. Hifill

#### 3.4. FGVC

## 4. Results

## 5. Discussion
