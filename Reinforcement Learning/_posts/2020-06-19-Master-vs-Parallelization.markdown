---
layout: post
title:  "Master vs Parallelization branch"
date:   2020-06-19 08:14:00 +0100
---
# Overnight training
Training on the master branch with the modified 100 maximum steps, returned no better results than with 50 maximum steps. This is because the high level critic is predicting the reward based on the number of steps taken by the low level. The paper called [Learning Multi-Level Hierarchies with Hindsight](https://arxiv.org/abs/1712.00948) must be checked to understand why each step, the high level takes the same number of actions. Why shouldn't each step be one action by the high level? Is it because they've used multiple levels and it was easier to implement like this?

Another aspect of the parallelization branch is that each step and for each environment, the process would calculate gradients N_ITERS times, sum them together and then place them on the queue. However, the master branch code applies the gradients after each calculation. This means each process must add to the queue the grads after each calculation.

# Debugging
After the change mentioned above, the graphs for low level look very similar when compared between the two branches. During training with the parallelization on, using the hyperparameters from the paper, each test, the robot behaves identical. There is no improvement and the high level goal does not move. Since the model works with one process, the next thing to try is to just slowly ramp up the amount of processes until it starts failing. Next test would be done with the following parameters:
~~~ python
X_PARAMS = {
    "ENV_NAME": 'FetchPickAndPlace-v1',
    "EP_LENGTH": 50,
    "GRAD_BATCH": 256,
    "TRAIN_BATCH": 2,
    "PROCESSES_COUNT": 2,
    "NUM_ENVS": 1,
    "GAMMA": 0.98,
    "LEARNING_RATE": 1e-4,
    "REPLAY_SIZE": int(1_000_000 / 2),  # total / processes_count
    "REPLAY_INITIAL": 1_000,
    "H": 10,
    "TEST_ITERS": 100,
    "N_ITERS": 50,
    "RANDOM_EPS": 0.3,
    "NOISE_EPS": 0.2,
    "TEST_LAMBDA": 0.3,
    "SEED": 1996
}
~~~

Already with those parameters, there are changes between each test. However, it takes so much longer for the low model to learn. 

# Transitions
Currently, the code stops adding transitions when the goal is reached. Therefore, at the beginning, high level transitions consists of one episode because the object is not moved. The HER paper adds all transitions, while the HAC paper add all transitions with -1 reward, except the last one which is rewarded with 0. The technique in the HAC paper makes sense because the episode is stopped when the goal is reached. Let's look at all the cases:
- arm is not touching the object. The transition is made of -1 rewards even though each goal has been set to last object position and goal is technically reached. Last sample reward is set to 0.
- arm is interacting with the object but not reaching the goal. The transition would contain samples where the movement that actually interacted with the object will be rewarded with -1 and the random movement at the end of the episode will be rewarded positively.
- arm is reaching the goal. The transition ends when the goal is reached and the reward is correctly given.

Problems:
- (first case) goal is reached, but negatively rewarded
- (second case) goal is reached, but only random movement rewarded

I still believe the change I made should improve the results.

# Low level training
The X_PARAMS needs to be tweaked to work similarly like the master branch, but faster. With 'TRAIN_BATCH' reduced to 1, the learning is very similar to master branch. It feels like this variable is setting a compromise between speed and effectiveness.

### Comparison results after 500 steps
1. TB = train batch
2. NE = number of environments
3. PC = process count

- Blue - TB1, NE1, PC1
- Pink - TB1, NE1, PC2
- Gray - TB1, NE2, PC2
- Red - TB2, NE2, PC2

![Low level accuracy](/assets/Master-vs-Parallelization/0_accuracy.png)
![Low level critic ref](/assets/Master-vs-Parallelization/0_critic_ref.png)
![Low level actor loss](/assets/Master-vs-Parallelization/0_loss_actor.png)
![Low level critic loss](/assets/Master-vs-Parallelization/0_loss_critic.png)
![High level accuracy](/assets/Master-vs-Parallelization/1_accuracy.png)
![High level actor loss](/assets/Master-vs-Parallelization/1_loss_actor.png)
![High level critic loss](/assets/Master-vs-Parallelization/1_loss_critic.png)

One thing I missed was the use of gamma. In HAC, gamma is set to 0 in the last sample of the episode. However, in HER, when creating the transitions, the value of gamma was unchanged. Moreover, in HER, no samples that have `done = True` are considered.

## Memory replay
### HAC
- Experience created by interacting with the environment
- Experience created if low level fails to reach goal
- Transition created at the end of the episode by setting goal to last state of the episode, rewards to -1, except for last sample, which has gamma and reward set to 0

### HER
- Transition created at the end of the episode by setting goal to last state of the episode, rewards recalculated for each sample, no sample with `done = True` considered

### My implementation
- Experience created by interacting with the environment
- Experience created if low level fails to reach goal
- Transition created at the end of the episode by setting goal to last state of the episode, rewards recalculated for each sample, stop adding any more samples (in this step) once goal is reached in one of the processed samples


With 4 processes in parallel, trying to reproduce the training done on master branch, it seems like it would take 7 hours to do 150k episodes. This is a huge improvement.

# Attempts
Training the model with HAC, HER and a couple of modifications. Training stopped after 50k episodes.
## Attempt 1
- Experience created by interacting with the environment
- Experience created if low level fails to reach goal
- Transition created at the end of the episode by setting goal to last state of the episode, rewards recalculated for each sample. `gamma` set to 0 for samples with `done = True` and for last samples in an episode.

### Results
Grasping of the object happened early on during the training. Only witnessed once during tests. The critic loss for both levels are very spiky. Low level successful, high level fails.

## Attempt 2
HAC implementation

### Results
Extremely similar to first Attempt. Loss is slightly bigger, however, the accuracy achieved is higher. High level still failed to learn. Same spikiness observed in the critic's loss.

## Attempt 3
HER implementation

### Results
The model fails to converge on both levels.

## Attempt 4
- Experience created by interacting with the environment
- Experience created if low level fails to reach goal
- Transition created at the end of the episode by setting goal to last state of the episode, rewards recalculated for each sample, stop adding any more samples (in this step) once goal is reached in one of the processed samples. Gamma is set to 0 for samples with `done = True` and for last samples in a transition.

### Results
The gripper moves between initial position and goal, gripping and releasing. This could be due to transitions created. This is an interesting pattern.
Results show failure to converge on both levels at first, however, a second run shows best results achieved so far.

## Attempt 5
- Experience created if low level fails to reach goal
- Transition created at the end of the episode by setting goal to last state of the episode, rewards recalculated for each sample, no sample with `done = True` considered

### Results
High level critic loss is a mess. Failure to converge at high level, while low level gets poor results.

## Overall
Attempt 1 and 4 show promising results. They will be tested more by training over a long period of time.

![Low level accuracy](/assets/Master-vs-Parallelization/t_0_accuracy.png)
![Low level actor loss](/assets/Master-vs-Parallelization/t_0_loss_actor.png)
![Low level critic loss](/assets/Master-vs-Parallelization/t_0_loss_critic.png)
![High level accuracy](/assets/Master-vs-Parallelization/t_1_accuracy.png)
![High level actor loss](/assets/Master-vs-Parallelization/t_1_loss_actor.png)
![High level critic loss](/assets/Master-vs-Parallelization/t_1_loss_critic.png)
 <div>
    <em>Attempt 1 - orange; Attempt 2 - green; Attempt 4 - red; Attempt 5 - pink</em>
 </div>{: .center-text}
