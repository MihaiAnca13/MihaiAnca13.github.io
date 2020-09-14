---
layout: post
title:  "Getting close"
date:   2020-09-14 08:30:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Training results
Before the weekend, the models have been trained using scaling threshold for the low level. Here are the graphs for almost 80k episodes:

|  |   |   |   |   |
:-:|:-:|:-:|:-:|:-:|
![Low level accuracy](/assets/Getting-close/0_accuracy.png) | ![Low level actor loss](/assets/Getting-close/0_loss_actor.png) | ![Low level critic loss](/assets/Getting-close/0_loss_critic.png) | ![Low level reward](/assets/Getting-close/0_reward.png)
![High level accuracy](/assets/Getting-close/1_accuracy.png) | ![High level actor loss](/assets/Getting-close/1_loss_actor.png) | ![High level critic loss](/assets/Getting-close/1_loss_critic.png) | ![High level accuracy](/assets/Getting-close/1_reward.png)

![Gif](/assets/Getting-close/run0.gif)

The accuracy graph reports 6%, however, when running the model_test.py script that renders a test environment, the accuracy using both levels is 37.5%. If the low level is replaced by hardcoded movements the accuracy achieved is 35%. This proves that high level can learn to adapt to the shortcomings of low level. 

There are two key questions here:
- why is the learning slowed down so heavily
- why is the accuracy reported so low

## Why is the accuracy reported so low
The model.py's test_net function is checking whether the target has been reached at least once. On the other hand, the accuracy graph is only incremented if the target is reached in the last frame. The accuracy is a list of 100 boolean values which get summed together for obtaining a percentage. Which way is preferred?

## Why is the learning slowed down
When using a 100% accuracy low level, the high level's training speed is increased. The hypothesis is that good experiences are more easily capitalized on since every time a greedy action is taken, it is more likely that the low level would actually follow it. 

Currently, if the environment goal is reached, the low level exits the loop and the high level gets rewarded. If the high level then generated the same target, it will get rewarded once more. This will most likely repeat for the remaining of the episode. In terms of behaviour, this is good because the model learns to hold the cube in the target position, However, the replay buffer gets filled with similar experiences. In other words, during learning, the most learned behaviour would be that of holding a position. This isn't necessary a problem, but a lot of time is wasted learning a simple behaviour instead of capitalizing on new useful experiences.

The solution might be to implement **prioritized experience replay**.

<!-- An issue with the way accuracy is currently calculated during training is that the variable is incremented multiple times per episode. For high level, accuracy should be incremented once per episode, and for the low level, the accuracy should be incremented once per target. Whether this only takes the last frame or not into account is less relevant. -->


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