---
layout: post
title:  "Debugging noise"
date:   2020-09-10 08:30:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Debugging
Yesterday, I've determined that the noise introduced by the low level is the reason why the high level cannot converge. In order to determine a way to fix it, noise has been added to the hard coded movement. The following noise makes the accuracy drop to 90% and high level training fail:
~~~ python
np.random.uniform([-0.05, -0.05, -0.05, -0.02], [0.05, 0.05, 0.05, 0.02])
~~~

By removing the noise added to the gripper, the accuracy only falls by 3-5%. If this manages to learn, it means that the inaccuracy in gripper is the problem. Therefore, the solution would be to reduce the threshold for the gripper state.

Another thing to try is to have the low level in the beginning, then transition to using the hardcoded version after a couple thousands of episodes to check whether the high level than learns.


<!-- ![Accuracy](/assets/Reduced-workspace-results/accuracy.png)
![Actor loss](/assets/Reduced-workspace-results/loss_actor.png)
![Critic loss](/assets/Reduced-workspace-results/loss_critic.png)

![Gif](/assets/Reduced-workspace-results/run0.gif) -->

<!-- ![Low level accuracy](/assets/Benefits-of-Normalization/0_accurac.png)
![Low level actor loss](/assets/Benefits-of-Normalization/0_loss_actor.png)
![Low level critic loss](/assets/Benefits-of-Normalization/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Benefits-of-Normalization/1_accuracy.png)
![High level actor loss](/assets/Benefits-of-Normalization/1_loss_actor.png)
![High level critic loss](/assets/Benefits-of-Normalization/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png) -->