---
layout: post
title:  "Initial state changes"
date:   2020-06-29 08:30:00 +0100
---
# Training
The models have been retrained for 34.8k episodes (per parallel process). The learning rates used were 1e-4 for both levels. The main difference compared to last successful train was the increased number of consecutive steps required for the low level to be accepted as successful (from 3 to 5). This change made the movements smooth and more predictable, however it have failed to learn to grasp.

# Changes
The initial state that starts with the object grasped sets the gripper to maximum gripping. However, the idea behind this initial state is to teach the robot how to grasp objects. Therefore, the gripper state would be set to fully open making it harder to the models to reach the goal at the beginning, but potentially assisting in learning how to grasp. New state:
~~~ python
qpos = np.array([ 4.04999929e-01,  4.80000000e-01,  3.49426365e-07, -8.28771666e-05,
                                       1.81293642e-10,  6.02888105e-02,  1.71759430e-02, -7.52886139e-01,
                                      -1.75626221e-02,  1.45514140e+00,  1.56300550e-02,  9.08737593e-01,
                                      -4.94641402e-03,  2.72635136e-02,  2.72635136e-02,  1.34465487e+00,
                                       7.49106352e-01,  5.03082600e-01,  9.99966890e-01, -6.10751607e-05,
                                       1.80792149e-03,  7.93383657e-03])
qvel = np.array([-2.49425853e-10, -2.40490521e-12,  6.93901452e-08, -2.05517261e-05,
                                      -4.42395981e-14,  7.22353302e-05,  4.04340980e-04,  1.09568143e-02,
                                      -8.86783313e-04, -1.24025604e-03,  1.05584176e-03, -9.58892063e-03,
                                      -8.43435272e-04,  5.02172921e-05, -4.99729109e-05,  7.34645056e-04,
                                       1.95034687e-03, -5.61790627e-03,  5.39857530e-02,  7.14284036e-04,
                                      -7.68953939e-05])
self.secondary_state = mujoco_py.MjSimState(act=None, qpos=qpos, qvel=qvel, time=0.4, udd_state={})
~~~

Another very important change would be to add a feature that would periodically save the model if the loss has increased since last compared to. This feature would have saved a copy of the model that learned how to grasp objects before over-fitting. An initial value of 1000 steps would be used.

This was achieved by adding the critic's loss to each high level train entry. The loss would then be compared to the loss last time the model was saved. Previous save would be erased and the new one would take it's place. Here's the code that handles this in the main loop:
~~~ python
high_critic_loss = models.apply_grads(grads)

if high_critic_loss is not None and high_critic_loss < last_loss and step - last_loss_step > PARAMS['SAVE_LOSS_CHECK']:
    last_loss = high_critic_loss
    last_loss_step = step

    common.delete_model(save_path, starts_with="loss_")

    base = "loss_%.3f_%d_" % (high_critic_loss, step)
    common.save_model(models, base, save_path)
~~~

After only 300 steps the arm managed to grasp the object and move it to the right position, however, that behaviour was quickly lost. Maybe different hyper-parameters could help capitalize that learning quicker.

## Results
Increasing the low level consecutive steps to 5 did not help with the jittery movements. Next attempt would be to not break if the goal is achieved. This means the low level would always take 10 actions for each goal created by the high level. Also, the high level will always create exactly 5 goals each episode.

# Reading to do / things to implement
- self adaptive parameters
- auto tuning of parameters
- entropy/lr in loss

Low level seems to learn how to follow instructions quite easily, however, problems occur when training continues. A way to find the sweet spot for stopping the training should be found. This would be represented by some sort of metric, different to how accuracy is currently calculated. This is because even though accuracy doesn't go over 60%, the end-effector follows the goal almost perfectly.