---
layout: post
title:  "Parallel processing"
date:   2020-06-15 08:15:00 +0100
---
# Results from training
Test/episode does not end when goal is reached. "Done" must be forcefully set to true.

Results after training for approximately 106500 episodes (20 hours):

![High level actor loss](/assets/Parallel-processing/1_loss_actor.png)
![High level critic loss](/assets/Parallel-processing/1_loss_critic.png)
![High level accuracy](/assets/Parallel-processing/1_accuracy.png)
![Low level critic loss](/assets/Parallel-processing/0_loss_critic.png)
![Low level actor loss](/assets/Parallel-processing/0_loss_actor.png)
![Low level accuracy](/assets/Parallel-processing/0_accuracy.png)

There are two colours because the training has been split into two sessions. The parameters of each model and the replay memory has been saved and reused after the first session.

## Note on HER example
The example about shooting a ball toward a net fails to cover the case when the ball is missed. Currently, during training, each time the ball is missed, the net is brought on top of the ball. This is like instant gratification regardless of the action taken. I couldn't find anything that speaks of this in the [HER](https://arxiv.org/pdf/1707.01495.pdf) paper, however, they did manage to solve this environment.

# Converting to parallel
Training took a long time and given that it failed to find an optimal policy for solving the task at high level, I want to convert the code so it runs more optimally, using all the processing power available. This would allow multiple configurations to be tested in a timely manner.

# HER implementation in PyTorch
I've found an implementation on GitHub, therefore, before starting to work on converting the code to parallel, I will have a look to see if I can find how the "missing of the ball" is handled. In the code found, the main method used is "future", rather than "final". This means that for a k=4, 75% of sample's goal are overwritten by the achieved goal. 

## Flow of the HER sample function:
1. Select samples based on k
2. Generate random offset for each sample selected
3. Replace goal with achieved goal of the (current sample+offset)
4. Calculate rewards for all samples selected

My code mimics exactly this behaviour. There are a two issues with the above method:
1. If the box is not moving at all, all samples would have a reward of 0, because the achieve goal would equal the current state in every frame
2. If the box is pushed by the robot, all samples between the last contact and the end of episode would have a reward of 0, which tells the robot that random movements give equal reward.

Regardless of these, the model still managed to learn. **Why?** **How?**