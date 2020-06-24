---
layout: post
title:  "Parallel processing 4"
date:   2020-06-18 09:24:00 +0100
---
# More Debugging
Yesterday, the last issue found was with the `test_lambda` variable and behaviour. Today, I'll start by debugging the code in the `agent`'s function `test`.

First bug found was to do with `GLOBAL_SEED`. PyTorch multiprocessing module recalls the file, which means all globally declared variable gets re-initialized. Therefore, `GLOBAL_SEED` is reset back to 0 every time a new process is spawned. This is fixed by using the process index plus a base seed as the random seed.

I've tracked the output of both normal call and test of the agent. The difference between each output of the test (no noise added) was significantly smaller. I am trying to mainly look at the new code that was added, since I know that the old code was working. Therefore, I will look into the gradients calculation and the optimization step next. While looking into this, I've realized that this behaviour is taking place even before any training. This means that there's something else broken.

I will try to do a test right after the model is initialized. There's also the idea of normalizing the input, however, I haven't seen it being applied before in RL. This test showed that the `Model` class does something weird initializing the networks. 

I've changed branches back to master and ran the test right after initialization of the `hac_agent`. The robot arm behaves the same. Setting `_max_episode_length` to 100 causes big changes in learning.

After changing back to the parallelization code, and modifying the config to:
~~~ python 
DEFAULT_PARAMS = {
    "ENV_NAME": 'FetchPickAndPlace-v1',
    "GRAD_BATCH": 256,
    "TRAIN_BATCH": 2,
    "PROCESSES_COUNT": 4,
    "NUM_ENVS": 2,
    "GAMMA": 0.98,
    "LEARNING_RATE": 1e-3,
    "REPLAY_SIZE": int(1_000_000 / 4), # total / processes_count
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
I've started a test run to see what happens after a couple hundreds of episodes. The low level doesn't reach the goal at all in 1300 episodes. The problem can be in the choice of hyper-parameters. The papers parameters are as follows:

~~~ python
"GRAD_BATCH": 128,
"PROCESSES_COUNT": 8,
"NUM_ENVS": 2,
"GAMMA": 0.98,
"LEARNING_RATE": 1e-3,
"REPLAY_SIZE": int(1_000_000 / 8), # total / processes_count
"N_ITERS": 40,
"RANDOM_EPS": 0.2,
"NOISE_EPS": 0.05,
~~~

While looking at parameters, I've came across this: "Input scaling: Neural networks have problems dealing with inputs of different magnitudes and therefore it is crucial to scale them properly. To this end, we rescale inputs to neural networks so that they have mean zero and standard deviation equal to one and then clip them to the range [−5,5]. Means and standard deviations used for rescaling are computed using all the observations encountered so far in the training". I've mentioned normalization earlier today, but now I have confirmation is required. This normalization will probably help solve the environment, but it would not fix the low level training right now. I want to fix that before making more modifications.

Another bug found with the seed. Each environment inside a process would have the same seed. Fixed it by using the following:
~~~ python
proc_nr = int(proc_name[1:])
    seed = proc_nr * PARAMS['NUM_ENVS']
    envs = [make_env(seed + n) for n in range(PARAMS['NUM_ENVS'])]
~~~

The following parameters should match the configuration on master:

~~~ python
ONE_PARAMS = {
    "ENV_NAME": 'FetchPickAndPlace-v1',
    "GRAD_BATCH": 256,
    "TRAIN_BATCH": 1,
    "PROCESSES_COUNT": 1,
    "NUM_ENVS": 1,
    "GAMMA": 0.98,
    "LEARNING_RATE": 1e-4,
    "REPLAY_SIZE": int(1_000_000 / 1),  # total / processes_count
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

Training shows that low level is unable to learn. However, this should be identical to master branch. After only 300 episodes:

![Low level accuracy](/assets/Parallel-processing-4/0_accuracy.png)
![Low level critic ref](/assets/Parallel-processing-4/0_critic_ref.png)
![Low level actor loss](/assets/Parallel-processing-4/0_actor_loss.png)
![Low level critic loss](/assets/Parallel-processing-4/0_critic_loss.png)