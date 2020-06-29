---
layout: post
title:  "Initial state changes"
date:   2020-06-29 08:30:00 +0100
---
# Training
The models have been retrained for 34.8k episodes (per parallel process). The learning rates used were 1e-4 for both levels. The main difference compared to last successful train was the increased number of consecutive steps required for the low level to be accepted as successful (from 3 to 5). This change made the movements smooth and more predictable, however it have failed to learn to grasp.

# Changes
The initial state that starts with the object grasped sets the gripper to maximum gripping. However, the idea behind this initial state is to teach the robot how to grasp objects. Therefore, the gripper state would be set to fully open making it harder to the models to reach the goal at the beginning, but potentially assisting in learning how to grasp.

Another very important change would be to add a feature that would periodically save the model if the loss has increased since last compared to. This feature would have saved a copy of the model that learned how to grasp objects before over-fitting. An initial value of 1000 steps would be used.