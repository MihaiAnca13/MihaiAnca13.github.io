---
layout: post
title:  "New metric"
date:   2020-06-30 08:30:00 +0100
---
# Changes
Continuing the discussion from yesterday, a new metric would be added for tracking the performance of each level. Every step, a variable would track, just like accuracy, how many times during a episode the goal has been achieved. The difference between this new metric and old accuracy is that each environment would have it's own stored value and the results would be averaged. Moreover, the variable would be reset each episode. In other words, `accuracy` is a running average, while the new metric is recalculated each episode.

Another change was to remove the `gamma = 0` for each experience stored that achieves the goal. This change made sense because the episode does not end when the goal has been reached, therefore reacing the goal is no longer the last action that must be taken. Gamma would still be set to 0 for the last sample in an episode (when `done = True`).

Even when movement seems perfect, the new metric reports a 1 in 4 successful attempts (at the low level) of reaching the goal. However, if the training continues, the jittery movements start to appear. I've reduced the thresholds by almost half, the hypothesis being that the a reduced allowance in movement would reduce the jittery movement as well.

Training commencing with the following parameters:
~~~ python
DEFAULT_PARAMS = {
    "ENV_NAME": 'FetchPickAndPlace-v1',
    "EP_LENGTH": 50,
    "GRAD_BATCH": 128,
    "TRAIN_BATCH": 2,
    "PROCESSES_COUNT": 4,
    "NUM_ENVS": 2,
    "GAMMA": 0.98,
    "HIGH_LEARNING_RATE": 1e-4,
    "LOW_LEARNING_RATE": 5e-5,
    "REPLAY_SIZE": int(1_000_000 / 4),  # total / processes_count
    "REPLAY_INITIAL": 1_000,
    "H": 10,
    "TEST_ITERS": 100,
    "N_ITERS": 40,
    "RANDOM_EPS": 0.2,
    "NOISE_EPS": 0.05,
    "TEST_LAMBDA": 0.3,
    "SEED": 1396,
    "SAVE_LOSS_CHECK": 1000
}
~~~

There are four cases which the models need to learn:
- pick up the object and move it in the air
- push the object to the position on the table
- move the grasped object to the potision in the air
- move the grasped object to the position on the table

With the current learning, at 6.6k episodes the model only managed to learn how to do the third and forth points mentioned above. However, the movements are very smooth. This means that the combination of `gamma != 0` and not stopping the episode early helped.

There might have been success in the early episodes of pushing the object to the goal position on the table, however, as mentioned yesterday, the model doesn't seem to capitalize on the good examples good enough. Adaptive learning rate might help here.