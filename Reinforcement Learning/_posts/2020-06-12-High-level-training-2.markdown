---
layout: post
title:  "High level training 2"
date:   2020-06-12 08:30:00 +0100
---
# Understanding the HER paper
In their [paper](https://arxiv.org/pdf/1707.01495.pdf), the Appendix explains the training procedure used. Moreover, the following paragraph has some important information about the capturing of transitions:

"So far the only additional goals we used for replay were the ones corresponding to the final state of the environment and we will call this strategy final. Apart from it we consider the following strategies:
- future — replay with k random states which come from the same episode as the transition being replayed and were observed after it
- episode — replay with k random states coming from the same episode as the transition being replayed
- random — replay with k random states encountered so far in the whole training procedure. All of these strategies have a hyperparameter k which controls the ratio of HER data to data coming from normal experience replay in the replay buffer."

What they mean by this paragraph, is that each sample in the transition being saved has as goal different states. In the default case (final), the goal of each sample is set to the last state encountered in that given episode. They've achieved 80% success with the "final" case on the pick and place environment, so I see no reason to change it for now.

Let's analyse the situation. The following assumption is made: "the arm is in the air above the table".
1. During the episode, the arm it's not even touching the object. By creating the HER transitions, the goal would be moved to the position of the object. All samples should, therefore, have a reward of 0 and gamma set to 0 because the goal is achieved in each. 
2. During the episode the object is moved. By creating the HER transitions, the goal would be moved to the last position of the object. The only sample having a reward of 0 would be the last one in transition because the movement created by the arm caused the object to move to the goal.
3. During the episode the object is moved in the goal. HER transitions would replicate the normal samples.
4. The episode starts with the object on top of the goal. No HER transitions would be created.

Currently, the code does not match the first point analysed. The question is, should those transitions even exist? I will try changing the code and retraining. If no improvements are made, I will remove them. I am afraid those would make the majority of training in the begginning, but without this, the rewards would be too sparse to learn anything.

After 6000 episodes, it looks like it could learn to do it. I've accidently stopped the learning, but I will restart it and leave it for longer.

The paper recommends multiple goals per episode. This is achieved by running multiple environments in parallel, so I'll look into multiprocessing and multiple environments next.

Another thing I've found in the Appendix, was that they were using the target network instead of the main one for testing. However, I would not implement this for now. They trained for a total of 160000 episodes. I want to leave my agent for training for at least 50000 to see if there is any improvement.

I've added cases that take care of saving the models and replay memory and of loading them on demand.
