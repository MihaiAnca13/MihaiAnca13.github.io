---
layout: post
title:  "Benefits of Normalization"
date:   2020-08-14 08:25:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Notes

After only 900 episodes it became clear that the normalization, with the same hyperparameters, is not beneficial for the training process. This is very surprising and shows that there might still be a bug. For now, however, the testing will continue without normalization.
 
![Low level accuracy](/assets/Benefits-of-Normalization/0_accuracy.png)
![Low level actor loss](/assets/Benefits-of-Normalization/0_loss_actor.png)
![Low level critic loss](/assets/Benefits-of-Normalization/0_loss_critic.png)
<!-- ![Low level reward](/assets/Normalization-3/0_reward.png) -->
![High level accuracy](/assets/Benefits-of-Normalization/1_accuracy.png)
![High level actor loss](/assets/Benefits-of-Normalization/1_loss_actor.png)
![High level critic loss](/assets/Benefits-of-Normalization/1_loss_critic.png)
<!-- ![High level accuracy](/assets/Normalization-3/1_reward.png) -->

## Changes
After disabling normalization, there have been a couple of changes to the transitions creation part of the `step` function. The last addition mentioned yesterday introduced a bug that did not create last step in high level transitions. Moreover, with the introduction of this skip mechanism, multiple tests should be run to check the benefits:
- no normalization,     gamma normal
- no normalization,     gamma set to 0 for last step
- normalization,        gamma normal
- normalization,        gamma set to 0 for last step

New code snippet:
~~~ python
        if len(goal_transition) > 1 and not goal_achieved:
            goal_transition[-1].reward = 0
            goal_transition[-1].gamma = 0
            exp_to_add = []
            for t in goal_transition:
                if level > 0:
                    t_next_state = common.unpack_obs_pnp(t.next_state)
                    t.goal = p_next_state.object_pos[0]

                    if goal_distance(t_next_state.object_pos[0], t.goal, self.high_thresholds):
                        t.reward = 0
                        # t.gamma = 0
                        exp = self.create_experience(level, state=t.state, action=t.action, next_state=t.next_state,
                                                     reward=t.reward, done=t.done, goal=t.goal, gamma=t.gamma,
                                                     object_grasped=t.object_grasped)
                        exp_to_add.append(exp)
                        break # See post on 2020-06-15 and 2020-08-13
                else:
                    t.goal = low_level_next_state
                    # if target is grasping and transition sample was grasping
                    if t.goal[-1] < 0.02 and t.object_grasped:
                        threshold = self.low_thresholds.copy()
                        threshold[-1] = 1
                    else:
                        threshold = self.low_thresholds

                    if goal_distance(t.goal, t.next_state, threshold):
                        t.reward = 0

                exp = self.create_experience(level, state=t.state, action=t.action, next_state=t.next_state,
                                             reward=t.reward, done=t.done, goal=t.goal, gamma=t.gamma,
                                             object_grasped=t.object_grasped)
                exp_to_add.append(exp)

            if len(exp_to_add) > 1:
                for exp in exp_to_add:
                    self.replay_memory[level].add(exp)
~~~

In other words, before this fix, the first and last samples in a transition were not being added to the replay buffer. 

Training up to 10k p.p.p. with no normalization show slight improvement when setting gamma to 0 for last sample of transitions. Testing with normalization will commence with the following initial values obtained:
~~~ python
Mean: tensor([ 1.3903e+00,  8.1682e-01,  5.8525e-01,  1.3737e+00,  7.6837e-01,
         4.4001e-01, -1.7280e-02, -4.4971e-02, -1.4228e-01,  3.1141e-02,
         3.1216e-02, -9.2897e-03,  1.3906e-02,  1.7598e-02, -1.0736e-04,
        -6.6355e-04, -1.7746e-03, -4.2460e-04,  1.0842e-03,  3.1859e-04,
         6.2063e-04,  9.1102e-04,  8.9260e-04,  2.7647e-04,  2.5556e-04,
         1.7736e-01,  7.3201e-02,  9.1552e-02,  8.4116e-03]) 
 Std: tensor([0.0010, 0.1167, 0.1050, 0.0833, 0.1098, 0.0581, 0.1056, 0.1354, 0.1266,
        0.0212, 0.0212, 0.6395, 0.2334, 0.5239, 0.0114, 0.0139, 0.0152, 0.0620,
        0.0401, 0.0444, 0.0119, 0.0144, 0.0131, 0.0067, 0.0067, 0.6059, 0.6105,
        0.5508, 0.0433]) 
 Sum: tensor([ 8.9655e+06,  5.2673e+06,  3.7740e+06,  8.8583e+06,  4.9548e+06,
         2.8374e+06, -1.1143e+05, -2.9000e+05, -9.1752e+05,  2.0082e+05,
         2.0130e+05, -5.9905e+04,  8.9675e+04,  1.1348e+05, -6.9234e+02,
        -4.2789e+03, -1.1444e+04, -2.7380e+03,  6.9915e+03,  2.0544e+03,
         4.0021e+03,  5.8747e+03,  5.7559e+03,  1.7828e+03,  1.6480e+03,
         1.1437e+06,  4.7203e+05,  5.9037e+05,  5.4242e+04]) 
 Sum_Squares: tensor([1.2406e+07, 4.3903e+06, 2.2799e+06, 1.2213e+07, 3.8849e+06, 1.2703e+06,
        7.3868e+04, 1.3118e+05, 2.3386e+05, 9.1645e+03, 9.1772e+03, 2.6381e+06,
        3.5241e+05, 1.7721e+06, 8.3846e+02, 1.2505e+03, 1.5036e+03, 2.4827e+04,
        1.0383e+04, 1.2735e+04, 9.1632e+02, 1.3517e+03, 1.1082e+03, 2.9198e+02,
        2.9195e+02, 2.5698e+06, 2.4384e+06, 2.0102e+06, 1.2538e+04]) 
 Count: tensor([6448497], dtype=torch.int32)
~~~