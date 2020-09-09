---
layout: post
title:  "High accuracy"
date:   2020-09-07 08:45:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Training
The training with both levels and reduced workspace failed. Therefore, a new round has been started with high level only and reduced workspace, in order to get a definitive accuracy result. This will form the baseline for the rest of the algorithms. At 70k episodes p.p.p., the accuracy has hit approximately 46%. After about 100k episodes, the accuracy stars decreasing, however, the loss is still decreasing.

Results after 337k episodes p.p.p. 57 hours of training):

![Accuracy](/assets/High-accuracy/accuracy.png)
![Actor loss](/assets/High-accuracy/loss_actor.png)
![Critic loss](/assets/High-accuracy/loss_critic.png)

![Gif](/assets/High-accuracy/run0.gif)

An accuracy of **92%** is reported after 50 renderings.

## Experiment 2
Another test that needs to be done is when the starting position is always in the air with the cube on the table. The goal will therefore be both on the table and in the air. However, this is not a priority.

*Note*: How can you train a model to determine whether an object is outside of range, and should therefore ignore it. When having a big pile of objects, if one rolls away, the model should not try to grasp it. This can be solved with the classification system, however, there might be cases where this is useful.

## Notes from Deep Learning Course (coursera)
Vectorization instead of for loops whenever possible will lead to 300 times faster compilation.

## Notes from Deep RL Bootcamp
- Huber loss instead of MSE for DQN
- Scaling epsilon from 1 to 0.01 over million of frames for DQN
- Prioritized experience replay based on Bellman error (update experiences that weren't used/seen that often)
- Noisy net parameters instead of epsilon greedy

## Realisation
The -H punishment is redundant. This is because when generating a target, the low level will get as close to it as possible without actually reaching it. This will then lead to the experience created for the high level to contain the last position reached, not the target generated. Therefore, not having any past experiences where out-of-reach targets were created will inhibit the high level from creating any further unreachable actions. Moreover, if generating a lot of negative rewards in the beginning, when the low level is still learning optimal behaviour, the high level will essentially get punished for random actions. This will lead to reduced exploration. However, very importantly, removing this from the algorithm would also remove the 'is_test' condition. This means the random action chance will have to be adjusted accordingly.

### Goal sampling problem
Workspace goes up to 0.2 meters height. However, the sample_goal functions adds a height offset between 0.14 and 0.19. This gets added to the current position of the cube (which sits on the table) of 0.035m. Fortunately, the threshold of 0.03 covers many of the cases when the goal is out of reach. 
~~~
max_height = 0.19+0.035
max_height - max_reach = 0.225 - 0.2 = 0.025 < 0.03
~~~

The hypothesis is that this affects the training accuracy reported on the graph, but not the final accuracy.

Moreover, the completely random actions sometime taken make the robot drop the cube. This has lead to an interesting scenario where the cube is dropped and the model has learned to pick it up again as quickly as possible.

### Question
What happens if the trained model is tested on a large workspace?

<!-- ![Low level accuracy](/assets/Benefits-of-Normalization/0_accuracy.png)
![Low level actor loss](/assets/Benefits-of-Normalization/0_loss_actor.png)
![Low level critic loss](/assets/Benefits-of-Normalization/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Benefits-of-Normalization/1_accuracy.png)
![High level actor loss](/assets/Benefits-of-Normalization/1_loss_actor.png)
![High level critic loss](/assets/Benefits-of-Normalization/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png) -->