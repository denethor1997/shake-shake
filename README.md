# Shake-Shake regularization of 3-branch residual networks

This repository contains the code for the paper [Shake-Shake regularization of 3-branch residual networks](https://openreview.net/forum?id=HkO-PCmYl&noteId=HkO-PCmYl). 

The code is based on [fb.resnet.torch] (https://github.com/facebook/fb.resnet.torch).

## Table of Contents
1. [Introduction](#introduction)
2. [Results](#results)
3. [Usage](#usage)
4. [Contact](#contact)

## Introduction
This method aims at helping computer vision practitioners faced with an overfit problem. The idea is to replace, in a 3-branch ResNet, the standard summation of residual branches by a stochastic affine combination. The largest tested model improves on the best single shot published result on CIFAR-10 by reaching 2.86% test error.

![shake-shake](https://s3.eu-central-1.amazonaws.com/github-xg/architecture3.png)

Figure 1: **Left:** Forward training pass. **Center:** Backward training pass. **Right:** At test time.

## Results
The base network is a 26 2x32d ResNet (i.e. the network has a depth of 26, 2 residual branches and the first residual block has a width of 32). "Shake" means that all scaling coefficients are overwritten with new random numbers before the pass. "Even" means that all scaling coefficients are set to 0.5 before the pass. "Keep" means that we keep, for the backward pass, the scaling coefficients used during the forward pass. "Batch" means that, for each residual block, we apply the same scaling coefficient for all the images in the mini-batch. "Image" means that, for each residual block, we apply a different scaling coefficient for each image in the mini-batch. The numbers in the Table below represent the average of 3 runs except for the 96d models which were run 5 times.

Forward | Backward | Level | 26 2x32d | 26 2x64d | 26 2x96d 
-------|:-------:|:--------:|:--------:|:--------:|:--------:|
Even	|Even	|n\a	|4.27	|3.76	|3.58
Even	|Shake	|Batch	|4.44	|-	|-
Shake	|Keep	|Batch	|4.11	|-	|-
Shake	|Even	|Batch	|3.47	|3.30	|-
Shake	|Shake	|Batch	|3.67	|3.07	|-
Even	|Shake	|Image	|4.11	|-	|-
Shake	|Keep	|Image	|4.09	|-	|-
Shake	|Even	|Image	|3.47	|3.20	|-
Shake	|Shake	|Image 	|3.55	|2.98	|**2.86**

Table 1: Average error rates of the last epochs (%) on CIFAR-10

## Usage 
0. Install [fb.resnet.torch] (https://github.com/facebook/fb.resnet.torch), [optnet](https://github.com/fmassa/optimize-net) and [lua-stdlib](https://github.com/lua-stdlib/lua-stdlib).
1. Download Shake-Shake
```
git clone https://github.com/xgastaldi/shake-shake.git
```
2. Copy the elements in the shake-shake folder and paste them in the fb.resnet.torch folder. This will overwrite 5 files (*main.lua*, *train.lua*, *opts.lua*, *checkpoints.lua* and *models/init.lua*) and add 3 new files (*models/shakeshake.lua*, *models/shakeshakeblock.lua* and *models/mulconstantslices.lua*).
3. You can train a 26 2x32d "Shake-Shake-Image" ResNet on CIFAR-10+ using

```
th main.lua -dataset cifar10 -nGPU 1 -batchSize 128 -depth 26 -shareGradInput false -optnet true -nEpochs 1800 -netType shakeshake -lrShape cosine -widenFactor 2 -LR 0.2 -forwardShake true -backwardShake true -shakeImage true
``` 

You can train a 26 2x96d "Shake-Shake-Image" ResNet on 2 GPUs using

```
CUDA_VISIBLE_DEVICES=0,1 th main.lua -dataset cifar10 -nGPU 2 -batchSize 128 -depth 26 -shareGradInput false -optnet true -nEpochs 1800 -netType shakeshake -lrShape cosine -widenFactor 6 -LR 0.2 -forwardShake true -backwardShake true -shakeImage true
```

A widenFactor of 2 corresponds to 32d, 4 to 64d, etc..

### Note
Changes made to fb.resnet.torch files:

*main.lua*  
Ln 17, 54-59, 81-88: Adds a log (courtesy of Sergey Zagoruyko)  

*train.lua*  
Ln 36-38 58-60 206-213: Adds the cosine learning rate function  
Ln 88-89: Adds the learning rate to the elements printed on screen  

*opts.lua*  
Ln 57-62: Adds Shake-Shake options  

*checkpoints.lua*  
Ln 15-16: Adds require 'models/shakeshakeblock' and require 'std'  
Ln 60-61: Avoids using the fb.renet.torch deepcopy (it doesn't seem to be compatible with the BN in shakeshakeblock) and replaces it with the deepcopy from stdlib  
Ln 67-81: Saves only the best model  

*init.lua*  
Ln 91-92: Adds require 'models/mulconstantslices' and require 'models/shakeshakeblock'  

The main model is in *shakeshake.lua*. The residual block model is in *shakeshakeblock.lua*. *mulconstantslices.lua* is just an extension of nn.mulconstant that multiplies elements of a vector with image slices of a mini-batch tensor.

## Contact
xgastaldi.mba2011 at london.edu  
Any discussions, suggestions and questions are welcome!

