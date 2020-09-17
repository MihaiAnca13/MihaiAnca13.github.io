---
layout: post
title:  "Data parallelism"
date:   2020-09-17 08:30:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Data Parallelism work
Reiterating the steps required:

## Steps required for transition to data parallelism
1. Mini Replay buffer for each agent
2. Networks copy shared and updated periodically - every 400 frames
3. Remove queue because experiences are added straight to the main replay buffer
4. Trim extra entries in buffer every 100 frames
5. ~~Move grad calculation to model.py~~
6. Sampling the buffer has to be done in parallel using a mp.Process and a queue of size 2. Therefore, each time a sample is consumed, another one is prepared in advance. Batch size is 256
7. Sending of data is done the same way, using a queue, that is emptied when filled. batch size = 50
8. ~~Gradient clipping between -1 and 1~~



<!-- |  |   |   |   |   |
:-:|:-:|:-:|:-:|:-:|
![Low level accuracy](/assets/Getting-close/0_accuracy.png) | ![Low level actor loss](/assets/Getting-close/0_loss_actor.png) | ![Low level critic loss](/assets/Getting-close/0_loss_critic.png) | ![Low level reward](/assets/Getting-close/0_reward.png)
![High level accuracy](/assets/Getting-close/1_accuracy.png) | ![High level actor loss](/assets/Getting-close/1_loss_actor.png) | ![High level critic loss](/assets/Getting-close/1_loss_critic.png) | ![High level accuracy](/assets/Getting-close/1_reward.png)

![Gif](/assets/Getting-close/run0.gif) -->


<!-- ![Accuracy](/assets/Reduced-workspace-results/accuracy.png)
![Actor loss](/assets/Reduced-workspace-results/loss_actor.png)
![Critic loss](/assets/Reduced-workspace-results/loss_critic.png)

![Gif](/assets/Reduced-workspace-results/run0.gif) -->