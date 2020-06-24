---
layout: post
title:  "Attempt 4 training"
date:   2020-06-22 08:14:00 +0100
---
# Attempt 4
- Experience created by interacting with the environment
- Experience created if low level fails to reach goal
- Transition created at the end of the episode by setting goal to last state of the episode, rewards recalculated for each sample, stop adding any more samples (in this step) once goal is reached in one of the processed samples. Gamma is set to 0 for samples with `done = True` and for last samples in a transition.

### Parameters
~~~ python
X_PARAMS = {
    "ENV_NAME": 'FetchPickAndPlace-v1',
    "EP_LENGTH": 50,
    "GRAD_BATCH": 256,
    "TRAIN_BATCH": 2,
    "PROCESSES_COUNT": 4,
    "NUM_ENVS": 2,
    "GAMMA": 0.98,
    "LEARNING_RATE": 1e-4,
    "REPLAY_SIZE": int(1_000_000 / 4),  # total / processes_count
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

### Results
After training for 53.5k episodes (per parallel process):  
![Low level accuracy](/assets/Attempt-4-training/0_accuracy.png)
![Low level actor loss](/assets/Attempt-4-training/0_loss_actor.png)
![Low level critic loss](/assets/Attempt-4-training/0_loss_critic.png)
![High level accuracy](/assets/Attempt-4-training/1_accuracy.png)
![High level actor loss](/assets/Attempt-4-training/1_loss_actor.png)
![High level critic loss](/assets/Attempt-4-training/1_loss_critic.png)