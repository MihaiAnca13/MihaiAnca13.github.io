---
layout: post
title:  "Overview results"
date:   2020-07-02 08:30:00 +0100
---
# Results

After training for 7.7k episodes (per parallel process):  
![Low level accuracy](/assets/Overview-results/0_accuracy.png)
![Low level actor loss](/assets/Overview-results/0_loss_actor.png)
![Low level critic loss](/assets/Overview-results/0_loss_critic.png)
![Low level reward](/assets/Overview-results/0_reward.png)
![High level accuracy](/assets/Overview-results/1_accuracy.png)
![High level actor loss](/assets/Overview-results/1_loss_actor.png)
![High level critic loss](/assets/Overview-results/1_loss_critic.png)
![High level reward](/assets/Overview-results/1_reward.png)
![Cumulative reward](/assets/Overview-results/reward_100.png)


![Final0](/assets/Overview-results/run0_0.gif)
![Final1](/assets/Overview-results/run0_1.gif)
![Final2](/assets/Overview-results/run0_2.gif)

## Conclusion

The movements achieved are smooth. The graphs show a continuous increase in loss, but also an increase in the cumulative reward (reward_100 graph). This is because the thresholds decrease with time and therefore it becomes harder to achieve the goals. The high level fails to learn all cases, however, this could be due to low training time. Jittery movements still appeared during training but did not persist this time. It seems like that problem is fixed, but high level training still needs to be improved.

# Tests
With a starting learning rate of 1e-3 for high level, the model outputs the same goal at every step after just one round of training. This test was done with a `grad_batch` of 256. A `grad_batch` of 128 produces the same results. 5e-3 with 128 returns similar results. 256 made no difference. 8e-3 gives the same results. However, looking at the results bellow, this suggests the model might benefit from different learning rates for critic and actor. This should replace the adaptable learning rate, rather than be an addition to it.

![High level actor loss](/assets/Overview-results/lr_1_loss_actor.png)
![High level critic loss](/assets/Overview-results/lr_1_loss_critic.png)

New default parameters:
~~~ python
DEFAULT_PARAMS = {
    "ENV_NAME": 'FetchPickAndPlace-v1',
    "EP_LENGTH": 50,
    "GRAD_BATCH": 128,
    "TRAIN_BATCH": 2,
    "PROCESSES_COUNT": 4,
    "NUM_ENVS": 2,
    "GAMMA": 0.98,
    "HIGH_CRITIC_LR": 5e-3,
    "HIGH_ACTOR_LR": 1e-4,
    "LOW_CRITIC_LR": 1e-4,
    "LOW_ACTOR_LR": 1e-4,
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

The previous behaviour where the high level generates the same goal is now gone with those values. Removing adapting learning rate reintroduced the jittery movements. Changes will be reverted.