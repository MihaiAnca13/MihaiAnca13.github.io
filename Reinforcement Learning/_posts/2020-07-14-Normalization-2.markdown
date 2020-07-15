---
layout: post
title:  "Normalization 2"
date:   2020-07-14 08:30:00 +0100
---
# Debugging continued

Today, training will be restarted and debugging will continue. Jittery movements must be removed once again. However, training for longer will tell if this problem gets removed with more experience.

In both levels, the critic seems to act similarly to before the change. However, the actor is reaching a local optimum after approximately 400 steps then starts to overfit.

## Comparing means
After 16 episodes, the normalizer is initialized with the following values for gripper position and state (on the left), while the actual mean is displayed on the right:
~~~ python
# normalizer |	actual mean
1.341			1.29
0.776			0.755
0.506			0.625
0.028			0.025
~~~

Since the actual mean is known for position and state, the normalizer object will have those values overwritten after the initialization. The hypothesis is that the random actions during the initialization does not cover the whole workspace of the robot, therefore, during training, there would be a lot of outliers that will throw off the training samples.

When it comes to low level actions, the means are very close to 0, which is equal to the actual mean. No need to overwrite these.

Another solution would have been to just run the initialize function for longer. This will be tried next if the current solution doesn't suffice.

The changes show an improvement in the low level because there are no more jittery movements. However, the high level still struggles to learn in the early episodes. Longer training should allow conclusions to be drawn.

## Results
After about 2.5k episodes per parallel process (p.p.p.), the accuracy of the low level starts decreasing. Here are the results for 5.4k episodes p.p.p.:

![Low level accuracy](/assets/Normalization-2/0_accuracy.png)
![Low level actor loss](/assets/Normalization-2/0_loss_actor.png)
![Low level critic loss](/assets/Normalization-2/0_loss_critic.png)
![Low level reward](/assets/Normalization-2/0_reward.png)
![High level accuracy](/assets/Normalization-2/1_accuracy.png)
![High level actor loss](/assets/Normalization-2/1_loss_actor.png)
![High level critic loss](/assets/Normalization-2/1_loss_critic.png)
![High level accuracy](/assets/Normalization-2/1_reward.png)

## Next attempt

For the next training, the learning rate has been increased and the initialization episodes for the normalizers have been increased to 100. Also, at the end of the training, the final mean and std will be printed, so that those values can be used as initializers for the `model_test` file. Also, this could be used as start values for training as well.

This got the high level stuck in a local optimum. Mean and std:
~~~ python
Mean: [1.2900000e+00  7.5500000e-01  6.2500000e-01  1.3553660e+00
  7.4276358e-01  4.2434770e-01  3.2224320e-02 -1.9820729e-02
 -9.7526029e-02  2.5000000e-02  2.7601941e-02 -1.5681272e-02
  5.3629256e-03 -1.1278855e-02  5.6629669e-04 -3.5478981e-04
 -1.1740174e-03  4.7888380e-04  5.2939315e-04  4.9625287e-05
 -2.9584349e-04  2.6876721e-04  4.2549949e-04  2.3311432e-04
  2.2919061e-04 -7.9099201e-03  1.1704305e-02  5.6447397e-04
  2.1972217e-05] 

 Std: [0.08852842 0.08067811 0.07165709 0.09798683 0.09483524 0.03626001
 0.13033696 0.11601525 0.0820585  0.01681347 0.01681633 0.5071758
 0.15565471 0.4840928  0.01473752 0.01505181 0.01536373 0.05751781
 0.04107248 0.05185914 0.0146377  0.01501699 0.01467267 0.00882183
 0.00885633 0.5788745  0.5747168  0.5729297  0.02881356] 

~~~