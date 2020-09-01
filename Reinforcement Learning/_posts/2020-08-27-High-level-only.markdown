---
layout: post
title:  "High level only"
date:   2020-08-27 08:30:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Code changes
The simulator `step` function has been adjusted to accept a target rather than an action. Therefore, at each step the end-effector will be moved exactly where the high level target is generated.

Moreover, the HAC agent's step function has been adjusted to skip all transition creation for the low level. All other mentions of low level have been removed throughout the code. What remains is a DDPG network trained across 4 processes with shared gradients. Speed on local computer is approximately 90 frames / second.

## Things to do
Figure out a way to extract data from runs from the supercomputer. This can be done with `scp user@ip:/path/to/file .` command after zipping runs and saves on the supercomputer.

One training was started on the cpu only just to compare the speed, however it came back with no data in the tensorboard. I will add more debugging messages for future runs.

# Results high level only
After training for only 30k on the local machine (4 parallel processes):

![Accuracy](/assets/High-level-only/accuracy.png)
![Actor loss](/assets/High-level-only/loss_actor.png)
![Critic loss](/assets/High-level-only/loss_critic.png)

![Gif](/assets/High-level-only/run0.gif)

<!-- ![Low level accuracy](/assets/Benefits-of-Normalization/0_accurac.png)
![Low level actor loss](/assets/Benefits-of-Normalization/0_loss_actor.png)
![Low level critic loss](/assets/Benefits-of-Normalization/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Benefits-of-Normalization/1_accuracy.png)
![High level actor loss](/assets/Benefits-of-Normalization/1_loss_actor.png)
![High level critic loss](/assets/Benefits-of-Normalization/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png) -->