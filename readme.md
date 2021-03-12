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

The dataset is composed of 13 small clips of subsea operations that were extracted from 5 videos. It also contains masks covering the overlay for each clip. Each clip has between 15 and 80 frames. The clips are this short because one of the inpainting techniques tested, without modifications, is not able to process long sequences, running out of memory.

Both the clips and masks are provided as individual PNG images with the same name in different folders. The masks are black and white images. White means inpaint here and black means keep the original content.

 <p align="center">
 <br>
   <img src="https://github.com/brunomsantiago/subsea_inpainting/raw/main/images/frame_plus_mask.png">
   <br><b>Figure 2 - A frame, its mask and the mask applied on the frame</b>
 </p>

The dataset was created by watching and downloading lots of youtube videos of subsea operations, mostly subsea inspections. Initially 43 videos were selected for further analysis and in the end only 5 was kept. These 5 videos are most similar to private videos I have seen on my work experience at Oil and Gas industry, however not exactly similar.

For download and more details about the folder structure of the dataset see [Subsea Inpainting Dataset at Kaggle Datasets](https://www.kaggle.com/brunomsantiago/subsea-inpainting-dataset).

## 3. Inpainting techniques tested

### 3.1. Frame by frame techniques

These techniques are designed to used on individual images. When used on videos they are applied frames by frame, without any data propagation from nearby frames, which may lead to inconsistencies between them.

#### 3.1.1. OpenCV

The first technique tested was OpenCV Navier-Stokes Image Inpainting (see [OpenCV inpainting tutorial](https://docs.opencv.org/master/df/d3d/tutorial_py_inpainting.html)) which is based on a [paper from 2001](https://ieeexplore.ieee.org/document/990497). It is a classic computer vision method, which have better results removing small defects, thin defects or larger defects on regular backgrounds.

I didn't have great expectations about this technique for removing overlay from subsea videos, but included it in the analysis as baseline. This baseline is useful both for inpainting results and processing time. More advanced techniques must have inpainting results that look much better.  Techniques ready for production should look for processing times as close a possible as opencv technique.

#### 3.1.2. Deepfill v1

Deepfill v1 ([paper](https://arxiv.org/abs/1801.07892), [code](https://github.com/JiahuiYu/generative_inpainting/tree/v1.0.0), [video](https://youtu.be/xz1ZvcdhgQ0)) and v2 ([paper](https://arxiv.org/abs/1806.03589), [code](https://github.com/JiahuiYu/generative_inpainting/tree/v2.0.0), [video](https://youtu.be/uZkEi9Y2dj4)) are inpainting techniques based on deep neural networks. From Deepfill v1 paper abstract:

> "... a new deep generative model-based approach which can not only synthesize novel image structures but also explicitly utilize surrounding image features as references during network training to make better predictions. The model is a feed-forward, fully convolutional neural network which can process images with multiple holes at arbitrary locations and with variable sizes during the test time"

Although Deepfill v1 and v2 have their official implementations on github, I didn't used them. I used v1 implementation from FGVC (see more details below), which uses Deepfill v1 as part of its processing pipeline.

#### 3.1.3. Hifill

Hifill ([paper](https://arxiv.org/abs/2005.09704), [code](https://github.com/Atlas200dk/sample-imageinpainting-HiFill), [video](https://youtu.be/Q7mX5Bstv7U)) is a inpainting technique similar to Deepfill, which is designed to work better on high resolution images, with larger roles and easier to adapt for new contexts (with few images). From the abstract:

> ... a Contextual Residual Aggregation (CRA) mechanism that can produce high-frequency residuals for missing contents by weighted aggregating residuals from contextual patches, thus only requiring a low-resolution prediction from the network. Since convolutional layers of the neural network only need to operate on low-resolution inputs and outputs, the cost of memory and computing power is thus well suppressed. Moreover, the need for high-resolution training datasets is alleviated. In our experiments, we train the proposed model on small images with resolutions 512x512 and perform inference on high-resolution images, achieving compelling inpainting quality. Our model can inpaint images as large as 8K with considerable hole sizes, which is intractable with previous learning-based approaches.

### 3.2. Frame propagation techniques

#### 3.2.1. FGVC

## 4. Testing Code

## 5. Results

## 6. Discussion
