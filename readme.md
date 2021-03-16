# Subsea Inpainting - Model Branch

This branch is completely different from the [master branch](https://github.com/brunomsantiago/subsea_inpainting).
It is used to store independent copies of third-party neural networks models (code and pre-trained weights).

I separated the branches because the main branch has some images animations, making it slow to download. You can clone just this branch (in Colab or Kaggle notebooks, for example) using the following command:

```
!git clone https://github.com/brunomsantiago/subsea_inpainting --branch models
```

## Avaiable models

### From [FGVC](https://github.com/vt-vl-lab/FGVC)
 - Deepfill v1 (image inpainting)
 - RAFT (Optical Flow Prediction)
 - Edge Completion

### From [Hifill](https://github.com/Atlas200dk/sample-imageinpainting-HiFill)
 - Hifill (image inpainting)
