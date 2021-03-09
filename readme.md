# Subsea Inpainting
### Removing overlays from subsea inspection videos using image/video inpainting <br><br>


## 1. Introduction

Oil and Gas industry generates thousands of hours of subsea inspection videos per day. The Oil and Gas operators also have millions of hours of archived video. <br>
These underwater videos are usually acquired by Remote Operated Vehicles (ROVs) and are crucial to the integrity management strategy of subsea assets, like pipelines and Wet Christmas Trees (WCTs), allowing Oil and Gas companies to operate their offshore fields safely. <br>

Most of these videos are have an "burned overlay", showing real time metadata like ROV coordinates, water depth and heading. On one hand the burned overlay guarantees the metadata is close to the image data even after operations like individual frame extraction and video trim. On the other hand, the overlay may obstruct important image data from users or confuse image processing algorithms. <br>

<p align="center">
<br>
  <img src="https://github.com/brunomsantiago/subsea_inpainting/raw/main/images/typical_subsea_frame.png">
  <br><b>Figure 1 - Typical Subsea Frame with overlay metadata</b>
</p>

This work is an experiment to see if current State of Art inpainting techniques are ready to remove overlays from subsea inspection videos. The techniques were tested "as is" (without any fine tuning for subsea inspection) and the results were evaluated visually.


## 2. The Dataset

The dataset started by watching and downloading youtube videos of subsea operations, mostly subsea inspections.<br>
Initially I watched more than fifty videos and downloaded forty-three for further analysis. In the end I selected five that were most similar to private videos I have seen on my work experience at Oil and Gas industry, however not exactly similar.<br>
From these five videos I extracted thirteen small clips. Each clip has between 15 and 80 frames. The clips are so short because one of the inpainting techniques tested, without modifications, is not able to process long sequences, running out of memory.<br>
I also created masks covering the overlay. These masks are black and white PNG images, white meaning inpaint here and black meaning keep the original content.<br>
Both the clips and masks are provided as individual PNG images with the same name in different folders. 
For download and more details regarding the dataset strucuture see [Subsea Inpainting Dataset at Kaggle Datasets](https://www.kaggle.com/brunomsantiago/subsea-inpainting-dataset).

## 3. Inpainting techniques tested

#### 3.1 OpenCV

#### 3.2. Deepfill v1

#### 3.3. Hifill

#### 3.4. FGVC

## 4. Results

## 5. Discussion
