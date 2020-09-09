---
layout: post
title:  "Checking differences"
date:   2020-09-04 08:35:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Why is master branch failing?
In order to find out why training fails on the master branch, a manual check of the branches will be done. This should show any major differences that could cause the issue. If this fails, another training with a different seed will be started. Certain techniques only have a low convergence chance. Hopefully, this is not the case.

No bugs were found when inspecting differences. The next step is to reduce the workspace again and check whether training improves.

Already, during training, after only 2k episodes, two spikes (1% accuracy) appear. This is very similar to how high-only training graph looked like. It's now expected that around 4k episodes, the accuracy will steadily increase.

Unfortunately, that was not the case. The training got stuck, as seen in the following graphs (17k ep p.p.p.):

![Low level accuracy](/assets/Checking-differences/0_accuracy.png)
![Low level actor loss](/assets/Checking-differences/0_loss_actor.png)
![Low level critic loss](/assets/Checking-differences/0_loss_critic.png)
![High level accuracy](/assets/Checking-differences/1_accuracy.png)
![High level actor loss](/assets/Checking-differences/1_loss_actor.png)
![High level critic loss](/assets/Checking-differences/1_loss_critic.png)

## Debugging with gifs
As highlighted in [Dreamer](https://github.com/danijar/dreamer), tensorboard supports images, including gifs, for logging. Moreover, tensorboardX supports videos directly. This feature will be added so it will help debug problem when training happens over night or on the supercomputer.

<!-- ![Accuracy](/assets/Expanding-workspace/accuracy.png)
![Actor loss](/assets/Expanding-workspace/loss_actor.png)
![Critic loss](/assets/Expanding-workspace/loss_critic.png) -->

<!-- ![Gif](/assets/Reduced-workspace-results/run0.gif) -->
 
<!-- ![Low level accuracy](/assets/Benefits-of-Normalization/0_accuracy.png)
![Low level actor loss](/assets/Benefits-of-Normalization/0_loss_actor.png)
![Low level critic loss](/assets/Benefits-of-Normalization/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Benefits-of-Normalization/1_accuracy.png)
![High level actor loss](/assets/Benefits-of-Normalization/1_loss_actor.png)
![High level critic loss](/assets/Benefits-of-Normalization/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png) -->