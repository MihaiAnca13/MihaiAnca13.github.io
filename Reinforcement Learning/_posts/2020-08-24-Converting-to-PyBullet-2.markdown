---
layout: post
title:  "Converting to PyBullet 2"
date:   2020-08-24 08:42:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Changes
Since last post, the environment and simulator were finalised. The main code was branched off develop and improvements made in `develop-normalizer` have been added. Git submodules have been fixed in the repository, now linking both manually created environments. 

Things left to do:
- update the `unpack` function since the object rotation has been made redundant. *Note*: maybe in the future try removing object and gripper angular velocity
- update check for grasping
- update rendering call in `test` function of the `model` module

After the script is running on local machine, pybullet must be installed on the super computer and the job.sh bash script updated.

<!-- ![Low level accuracy](/assets/Benefits-of-Normalization/0_accurac.png)
![Low level actor loss](/assets/Benefits-of-Normalization/0_loss_actor.png)
![Low level critic loss](/assets/Benefits-of-Normalization/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Benefits-of-Normalization/1_accuracy.png)
![High level actor loss](/assets/Benefits-of-Normalization/1_loss_actor.png)
![High level critic loss](/assets/Benefits-of-Normalization/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png) -->