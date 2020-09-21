---
layout: post
title:  "Data parallelism"
date:   2020-09-17 08:30:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Data Parallelism work
Reiterating the steps required:

## Steps required for transition to data parallelism
1. ~~Mini Replay buffer for each agent~~
2. ~~Networks copy shared and updated periodically - every 400 frames~~
3. ~~Remove queue because experiences are added straight to the main replay buffer~~
4. Trim extra entries in buffer every 100 frames
5. ~~Move grad calculation to model.py~~
6. ~~Sampling the buffer has to be done in parallel using a mp.Process and a queue of size 2. Therefore, each time a sample is consumed, another one is prepared in advance. Batch size is 128~~
7. ~~Sending of data is done using a parallel process that just compares the current buffer count against its capacity. batch size = 50~~
8. ~~Gradient clipping between -1 and 1~~


## Debugging
Upon starting a test run, there were a couple of issues:
1. The `count` of the buffers was not being shared. This was due to a new tensor being created instead of the value being overwritten upon calling the `empty` function.
2. The weights are not shared properly even though the `sync` function is called periodically. Or, the training is not working due to bad samples being sampled. 

By printing the weights of the first neuron of the first layer, it is clear that the weights don't get updated. It turns out the weights of the network don't change. Somehow the loss is extremly low and so the gradients are really small, causing no changes. How does actor loss go down without an improvement in policy?


<!-- |  |   |   |   |   |
:-:|:-:|:-:|:-:|:-:|
![Low level accuracy](/assets/Getting-close/0_accuracy.png) | ![Low level actor loss](/assets/Getting-close/0_loss_actor.png) | ![Low level critic loss](/assets/Getting-close/0_loss_critic.png) | ![Low level reward](/assets/Getting-close/0_reward.png)
![High level accuracy](/assets/Getting-close/1_accuracy.png) | ![High level actor loss](/assets/Getting-close/1_loss_actor.png) | ![High level critic loss](/assets/Getting-close/1_loss_critic.png) | ![High level accuracy](/assets/Getting-close/1_reward.png)

![Gif](/assets/Getting-close/run0.gif) -->


<!-- ![Accuracy](/assets/Reduced-workspace-results/accuracy.png)
![Actor loss](/assets/Reduced-workspace-results/loss_actor.png)
![Critic loss](/assets/Reduced-workspace-results/loss_critic.png)

![Gif](/assets/Reduced-workspace-results/run0.gif) -->