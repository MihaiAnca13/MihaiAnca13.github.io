---
layout: post
title:  "High only training"
date:   2020-09-01 08:30:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Results high level only training
After training for 441k episodes p.p.p on the local machine:

![Accuracy](/assets/High-only-training/accuracy.png)
![Actor loss](/assets/High-only-training/loss_actor.png)
![Critic loss](/assets/High-only-training/loss_critic.png)

![Gif](/assets/High-only-training/run0.gif)

## Supercomputer
Unfortunately, the supercomputer scripts are still pending in the queue.

# Interpreting results
- The rendering shows that the cube must have friction added so it won't slide
- Having both starting positions and goals on the table seems to do more harm

## Cube friction
When rendering just the simulator, the cube has the correct amount of friction. Regardless, when creating the gif, it's clear that the friction has to be increased. A value of 10, replaced in the URDF file seems to do the trick:
~~~ xml
<link name="cube">
    <contact>
      <lateral_friction value="10.0"/>
      <rolling_friction value="0.0"/>
      <contact_cfm value="0.0"/>
      <contact_erp value="1.0"/>
    </contact>
~~~

## Start positions
Hypothesis: Having both starting positions and goal positions on the table surface leads to local optimum. The model doesn't learn how to grab and lift because it's easier to drop or push the object in the right location. Removing the goals on the table will potentially help with that. Possible foreseen problem is an inability to learn to grab which will lead to a huge actor loss from which the model won't be able to recuperate.

Changes:
~~~ python
# FROM
if self.np_random.random() < 0.5:
    self.goal_pos[1] += 0.2  # height offset
# TO
if self.np_random.random() < 1:
    self.goal_pos[1] += self.np_random.uniform(0.15, 0.3)  # height offset
~~~

## Further changes
Hypothesis: Reducing the workspace will result in faster convergence. Having a simpler/smaller environment will result in the robot arm interacting with the cube more often which will lead to better experiences. New reduced workspace:
~~~ python
# model bounds
state_bounds_np = np.array([0.075, 0.0865, 0.075, .02])
state_bounds = torch.FloatTensor(state_bounds_np.reshape(1, -1)).to(self.device)
state_offset_np = np.array([-0.025, 0.1135, -0.605, .02])
state_offset = torch.FloatTensor(state_offset_np.reshape(1, -1)).to(self.device)
state_clip_low = np.array([-0.1, 0.027, -0.68, 0.])
state_clip_high = np.array([0.05, 0.2, -0.53, 0.04])

# goal sampling
self.goal_pos = self.np_random.uniform([-0.08, 0.03499, -0.66], [0.048, 0.0349, -0.55])

if self.np_random.random() < 1:
    self.goal_pos[1] += self.np_random.uniform(0.14, 0.19)  # height offset

# object pos start
object_pos = self.np_random.uniform([-0.07, 0.03499, -0.65], [0.047, 0.0349, -0.56])
~~~

### Training
5k episodes in and the arm hasn't reached the goal not even once.
<!-- ![Low level accuracy](/assets/Benefits-of-Normalization/0_accurac.png)
![Low level actor loss](/assets/Benefits-of-Normalization/0_loss_actor.png)
![Low level critic loss](/assets/Benefits-of-Normalization/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Benefits-of-Normalization/1_accuracy.png)
![High level actor loss](/assets/Benefits-of-Normalization/1_loss_actor.png)
![High level critic loss](/assets/Benefits-of-Normalization/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png) -->
