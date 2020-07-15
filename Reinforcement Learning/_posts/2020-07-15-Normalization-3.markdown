---
layout: post
title:  "Normalization 3"
date:   2020-07-15 08:31:00 +0100
---
# Debugging continued

The `5e-3` learning rate proves to be too large still. The reason behind this could be due to the assumption that the low level is perfectly trained when generating actions for the high level. However, the fact that low level accuracy no longer stays high is a new problem. In only 500 episodes p.p.p., the low level manages to reach 80% accuracy. On the other side, the high level did not manage to learn any behaviour whatsoever. 

## Next steps

The values obtained last training for mean and std will be used as initial values for the normalizer. Unfortunately, those values will be quickly overwritten with the next update since the sum and count are initialized with 0. The `sum`, `count` and `sq_sum` will be printed next time and used instead.

![Bug found](/assets/Common/bug-stop.png){: .center-image}

The normalizer object does not get shared between processes. At the end of a short run, I've noticed the count was equal to 5000. This represents the number of frames analysed after the initialization. Passing model's networks works because of the `share_memory` function invoked in the constructor of the `Model`. However, the normalizer class does not benefit of this function. 

### Solution
The normalizer class was converted to use tensors instead of numpy arrays. This allows for the `share_memory_` function to be called for each variable. This way, each HAC object has access to the same memory locations. It was very important to convert all arguments to each `Normalizer` functions to a Tensor with float32 type. Otherwise, the default type is float64, which is considered a `Double` and the execution will halt with an error.

## Results

After running for 6500 episodes p.p.p.:
~~~ python
Mean: tensor([ 1.2185e+00,  6.2801e-01,  5.4912e-01,  1.3458e+00,  7.1972e-01,
         4.2815e-01,  1.2674e-01,  8.9719e-02, -1.2454e-01,  2.4814e-02,
         2.4780e-02,  1.2338e-02,  1.9432e-02,  1.6992e-01,  2.5304e-03,
         1.8472e-03, -1.2703e-03,  6.5688e-04,  2.8408e-04,  2.8229e-03,
        -2.5004e-03, -2.4654e-03,  1.8153e-04,  6.0470e-05,  1.1527e-04,
        -6.8568e-02, -1.1854e-01,  3.0074e-02, -3.2142e-03]) 
 Std: tensor([0.1452, 0.2310, 0.0940, 0.0499, 0.1275, 0.0631, 0.1733, 0.2244, 0.1197,
        0.0222, 0.0222, 0.4757, 0.2423, 0.7604, 0.0165, 0.0172, 0.0189, 0.0402,
        0.0401, 0.0539, 0.0166, 0.0175, 0.0172, 0.0071, 0.0071, 0.7372, 0.7187,
        0.6824, 0.0471]) 
 Sum: tensor([ 3.1139e+06,  1.6049e+06,  1.4033e+06,  3.4392e+06,  1.8392e+06,
         1.0941e+06,  3.2387e+05,  2.2927e+05, -3.1825e+05,  6.3410e+04,
         6.3325e+04,  3.1529e+04,  4.9658e+04,  4.3422e+05,  6.4665e+03,
         4.7204e+03, -3.2462e+03,  1.6786e+03,  7.2595e+02,  7.2139e+03,
        -6.3898e+03, -6.3002e+03,  4.6389e+02,  1.5453e+02,  2.9456e+02,
        -1.7522e+05, -3.0293e+05,  7.6853e+04, -8.2137e+03]) 
 Sum_Squares: tensor([3.8482e+06, 1.1443e+06, 7.9315e+05, 4.6348e+06, 1.3653e+06, 4.7862e+05,
        1.1782e+05, 1.4928e+05, 7.6229e+04, 2.8289e+03, 2.8241e+03, 5.7855e+05,
        1.5095e+05, 1.5514e+06, 7.1486e+02, 7.6286e+02, 9.1978e+02, 4.1221e+03,
        4.1160e+03, 7.4337e+03, 7.2039e+02, 8.0019e+02, 7.5850e+02, 1.3009e+02,
        1.3013e+02, 1.4009e+06, 1.3558e+06, 1.1925e+06, 5.7017e+03]) 
 Count: tensor([2555467], dtype=torch.int32) 
~~~

![Low level accuracy](/assets/Normalization-3/0_accuracy.png)
![Low level actor loss](/assets/Normalization-3/0_loss_actor.png)
![Low level critic loss](/assets/Normalization-3/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Normalization-3/1_accuracy.png)
![High level actor loss](/assets/Normalization-3/1_loss_actor.png)
![High level critic loss](/assets/Normalization-3/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png)

Analysing the graphs, it looks like the low level is struggling to maintain the learned behaviour, which results in poor accuracy for high level as well. The problem appears after 2k episodes. Looking at the renderings of the final weights, the high level seems to create the same point regardless of the state. This indicates that the high level model has overfitted on a local optimum.

Thresholds and learning rate at this step should be:

~~~ python
Learning rate both levels: 8e-5
Thresholds: 0.041
~~~

Comparing the calculated mean with the actual state mean, the numbers are quite different. This indicates that the model overfitted quite early (possible at 2k episodes), and all further samples pushed the mean towards that point. Unfortunately, this means that the values obtained cannot be used as a starting point. 

It's quite possible, however improbable in this case, that it's just a bad seed. Training will be restarted with a new seed.

There are two possibilities:
- low level training goes wrong at some point and the high level fails to learn any further
- the high level training goes wrong and the low level cannot keep up because the chosen point is outside the workspace (this has been prevented as much as possible)

Comparing previous graphs of low level training with current behaviour, I observed that there are many more spikes in the accuracy. The behaviour before normalization was much more constant. It's likely that there still is a bug that messes the training.

At 200 episodes, the high level creates the same target at every loop. This was observed during test rendering. At 400, the high level creates the target in a corner, however the gripper state changes. At 600, the low level perfectly follows the instructions. The high level starts to create more varied actions, but still concentrated in the same area. It's very clear that the high level struggles to create meaningful targets. The problem is that the graphs don't show anything meaningful. The critic's and actor's loss continue to decrease even though the behaviour is highly inadequate. 