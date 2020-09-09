---
layout: post
title:  "Pre-trained Combined"
date:   2020-09-09 08:45:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Debugging combined version
Given that there was a difference in training between high level only and the combined version, the next thing to try is to pre-train low level. This will re-create the same experiences for high level given that low level acts exactly as it should from the very beginning. If that is not the case, it means there is a bug and this should help finding it. In first 4-5k episodes, the accuracy should start to increase a tiny bit.

### Note for readers
Last entry was updated with graphs of the long training. Accuracy of 92% achieved!

<!-- ![Accuracy](/assets/High-accuracy/accuracy.png)
![Actor loss](/assets/High-accuracy/loss_actor.png)
![Critic loss](/assets/High-accuracy/loss_critic.png)

![Gif](/assets/High-accuracy/run0.gif) -->

<!-- ![Low level accuracy](/assets/Benefits-of-Normalization/0_accuracy.png)
![Low level actor loss](/assets/Benefits-of-Normalization/0_loss_actor.png)
![Low level critic loss](/assets/Benefits-of-Normalization/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Benefits-of-Normalization/1_accuracy.png)
![High level actor loss](/assets/Benefits-of-Normalization/1_loss_actor.png)
![High level critic loss](/assets/Benefits-of-Normalization/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png) -->