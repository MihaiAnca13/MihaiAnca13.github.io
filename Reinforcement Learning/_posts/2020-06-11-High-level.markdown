---
layout: post
title:  "High level training"
date:   2020-06-11 08:47:00 +0100
---
# Changes
I've added an accuracy meter, which tracks if, at each step, the goal has been reached. This helps stop training if the accuracy gets high enough. After training for about 1200 episodes, the accuracy of low level got to 57%. The training was done using a slightly lower learning rate, thinking it would facilitate the high level.

I've also managed to change the environment so that it closes the window after rendering. Not doing so, caused the window to display the message "Not responding".

By starting rendering after 300 episodes, I've noticed the "high level success" message was being displayed without the object getting anywhere near the target. At the same time, I was thinking of using a prioritized replay buffer, because the success of moving the object to the goal is so rare, that I feel like the network can capitalize on learning from it multiple times. I will hold on to this change until I can get the network to generate an action that moves the box purposefully.

From looking at renderings, I've observed that the high level was succeeding only when the goal was generated on top of the box. I've added an "is_test" variable that disables the noise/random action and gets triggered randombly based on "lamba". This is exactly the same as the original implementation. I didn't initially include it in the code because I didn't see its value, however, I now believe it can help the high level generate more realistic targets. Looking at renderings, I've realised that a lot of the goals are very high/at the edge of the workspace. I hope this will fix it.

Training commenced with the following params:
~~~ python
GAMMA = 0.99
BATCH_SIZE = 256
LEARNING_RATE = 1e-4
REPLAY_SIZE = 1_000_000
REPLAY_INITIAL = 1_000
H = 20
TEST_ITERS = 100
N_ITERS = 50
RANDOM_EPS = 0.3
NOISE_EPS = 0.2
TEST_LAMBDA = 0.3
~~~

After 3000 episodes, the agent learned that by moving randomly around the point that the high level generated, it has higher chances of reaching it. This is unwanted behaviour and should be dealt with. One way I can think of is by stopping the learning very early on when the movements are smooth. Another solution would be to count the number of consecutive steps in which the end-effector is within the target's range and only reward fix movements. After 3000 episodes, the accuracy of low level seems to cap at 80%.

I've also realised that each episode ends after 50 steps, which is not enough for 3 complete actions. I've reduced H to 10 to see the changes. The accuracy of the low level took a big hit, however, the movements are now smooth and exact. Also, after the changes, the high level started creating goals far away from the table. This got mitigated after longer training.

Training it for 1h 30mins, equivalent of 7400 episodes, did not improve the high level.

Hindsight experience replay ([HER](https://arxiv.org/pdf/1707.01495.pdf)) paper exaplains through a basic bit flipping environment example, how "the real problem is not in lack of diversity of states being visited, rather it is simply impractical to explore such a large state space". The solution, which was implemented in the code, "moves" the goal so the network believes it stumbled across a good example. For example, shooting a ball into a net and missing could help learn the task by simply "moving" the net where the ball ended. In our case, this doesn't work because by moving the goal where the object is, the network still had no interference with the object. If the object would be tied to the end-effector, this task would be possible to learn. However, this same task was succesfully learned using HER+DDPG. How they bypassed that problem: "to make exploration in this task easier we recorded a single state in which the box is grasped and start half of the training episodes from this state". They've also stated that "We have later discovered that training is possible without this trick if only the goal positionis sometimes on the table and sometimes in the air".



