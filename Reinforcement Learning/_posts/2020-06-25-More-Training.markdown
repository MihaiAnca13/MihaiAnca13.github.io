---
layout: post
title:  "More training"
date:   2020-06-25 08:14:00 +0100
---
# Results
After training for 23k episodes (per parallel process):  
![Low level accuracy](/assets/More-training/0_accuracy.png)
![Low level actor loss](/assets/More-training/0_loss_actor.png)
![Low level critic loss](/assets/More-training/0_loss_critic.png)
![High level accuracy](/assets/More-training/1_accuracy.png)
![High level actor loss](/assets/More-training/1_loss_actor.png)
![High level critic loss](/assets/More-training/1_loss_critic.png)

![Run 0 - 0](/assets/More-training/run0_0.gif)
![Run 0 - 1](/assets/More-training/run0_1.gif)
![Run 0 - 2](/assets/More-training/run0_2.gif)

# Discussion
If the episode is ended when the goal is achieved, the robot learns that it can still get the reward by pushing the box on top of the goal. This means that the goal is not truly reached, because the object would not be staying on the goal. Manual control is desired, therefore early stop of the episode is not desirable. Not stopping the episode helps the robot arm understand that controlling the box gets better reward in the long run. Low level could mimic this as well but it's not necessary since the 3-step check makes sure the goal is achieved and maintained.

Currently, in the `step` function, if the goal is achieved (at any level), the loop is stopped and the function exits. This means that once the high level goal is achieved, there would only be one step transitions being added to the replay buffer. However, if the loop is never stopped, each step the high level takes 10 decisions, and low level a total of 100 actions (for H = 10). This guarantees that an equal number of transitions are being added each step. On the other hand, the model should be able to take as many high level actions as possible in order to optimize the movement of the arm. If the low level loop is stopped once the goal is achieved, that allows the high level to generate a new one. If the high level loop is stopped once the overall goal is achieved, then we run into the problem discussed in the previous paragraph. The hypothesis is that each level should be handled separately. The high level never gets stopped, however, the low level stops once the generated goal has been achieved for 3 consecutive steps.

One question rises when dealing with one step transitions: should those be avoided, or is there something the model can learn from them? The action taken this step did not lead to the goal. The action taken last step was the one that achieved the goal, therefore this one step transition would promote random actions. 

Following the parameters recommended by the 'HER' paper, the learning rate is set to 1e-3. However, with that rate, the low level always fails to learn. Since each level have separate optimizers, setting different learning rates is easy. High level would be 1e-3 and low level 1e-4.

To facilitate training, the distance thresholds are quite high. However, to increase precision, those thresholds could be reduced over time. Sort of like epsilon in e-greedy algorithms.

When the goal is achieved from start, the high level generates an action that the low level ignores. The arm moves constantly in the opposite direction. Is this causing issues in training? What could cause this?

