---
layout: post
title:  "Debugging noise 2"
date:   2020-09-11 08:30:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Debugging
A training has been started with both levels enabled, no pre-trained weights on a reduced workspace environment. Low level threshold has been reduced for the gripper state. After 65k episodes p.p.p. (see images bellow), the high level has reached approximately 4% accuracy. However, during test renderings, it is clear that it has learned that picking up the box and moving it to the target gives good rewards. The problem seems to be the low level movement. The accuracy of the low level seems to be around 90%. However, the movements are not smooth, therefore the grasped object gets thrown around easily due to the sudden stops (inertia keeps the box moving in the same direction). 

![Low level accuracy](/assets/Debuggin-noise-2/0_accuracy.png)
![Low level actor loss](/assets/Debuggin-noise-2/0_loss_actor.png)
![Low level critic loss](/assets/Debuggin-noise-2/0_loss_critic.png)
![Low level reward](/assets/Debuggin-noise-2/0_reward.png)
![High level accuracy](/assets/Debuggin-noise-2/1_accuracy.png)
![High level actor loss](/assets/Debuggin-noise-2/1_loss_actor.png)
![High level critic loss](/assets/Debuggin-noise-2/1_loss_critic.png)
![High level accuracy](/assets/Debuggin-noise-2/1_reward.png)

The solution would be to quantify the "smoothness" of movements in the reward function. Given that the low level receives a higher reward for reaching the target as fast as possible, smooth movements would only slow it down. The jerky stop at the end could be removed by making the threshold very small. The downside would be that the learning speed would heavily be affected. One option would be to re-introduce scaling thresholds.

Before any change, there's one test that can verify the learned behaviour of the high level. When testing the model, the low level can be replaced by the hard coded movement. This would give an accurate reading of the accuracy. Accuracy: 47.5%! Here's part of the run:

![Gif](/assets/Debuggin-noise-2/run0.gif)

## First attempt
Re-introducing scaling thresholds for low level only.

### In the meantime
Taking screenshots of the tensorboard graphs it's a repetitive time-consuming action. Therefore, I'll automate it using matplotlib. Example of result:

![Example](/assets/Debuggin-noise-2/example.png)

<!-- ![Accuracy](/assets/Reduced-workspace-results/accuracy.png)
![Actor loss](/assets/Reduced-workspace-results/loss_actor.png)
![Critic loss](/assets/Reduced-workspace-results/loss_critic.png)

![Gif](/assets/Reduced-workspace-results/run0.gif) -->

<!-- ![Low level accuracy](/assets/Benefits-of-Normalization/0_accuracy.png)
![Low level actor loss](/assets/Benefits-of-Normalization/0_loss_actor.png)
![Low level critic loss](/assets/Benefits-of-Normalization/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Benefits-of-Normalization/1_accuracy.png)
![High level actor loss](/assets/Benefits-of-Normalization/1_loss_actor.png)
![High level critic loss](/assets/Benefits-of-Normalization/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png) -->