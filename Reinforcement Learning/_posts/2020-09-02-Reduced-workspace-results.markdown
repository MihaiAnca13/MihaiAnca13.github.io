---
layout: post
title:  "Reduced workspace results"
date:   2020-09-02 08:30:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Results high level only training
After training for 72k episodes p.p.p on the local machine:

![Accuracy](/assets/Reduced-workspace-results/accuracy.png)
![Actor loss](/assets/Reduced-workspace-results/loss_actor.png)
![Critic loss](/assets/Reduced-workspace-results/loss_critic.png)

![Gif](/assets/Reduced-workspace-results/run0.gif)

Even though the workspace is reduced and the robot arm interacts with the box often, the model did not learn to grasp. One issue could be the fact that the grasping is always happening after the movement is complete. Reversing this might help.

Also, in the gif the movements are very small when the goal is in the air and cube on the table. This could be due to a bug that punishes the model with -10 reward for not achieving the targets. A debug message will be added to the next training to make sure that is not happening.

![Bug found](/assets/Common/bug-stop.png){: .center-image}

Early episodes show that this is indeed the case:

![Punishment](/assets/Reduced-workspace-results/punishment.png){: .center-image}

By the time the first test rending is starting, the model has learned that moving (experimenting) is very bad. This is clearly a bug because the low level is using perfect movements and due to a reduced workspace, should always achieve the target. Here is an example of it failing:

~~~ python
target = [0.05 0.027 -0.53 0]
current = [0.049 0.028 -0.531 0.037]
~~~

The problem is not the simulator control, but the fact that sometimes the cube is grasped. When creating transitions this is taken into account, however, it hasn't been handled accordingly here. However, in the example above that was not the case, since when the cube is grasped the gripper state equals 0.22 approximately. This change seems to have completely removed the "Punish" debug message!

## While Training
While training runs in the background, I've decided to do some refactoring of the code in order to optimize it. 

### Type hints
"Using type hints for performance optimizations is left as an exercise for the reader" - documentation 

Example of type hints:
~~~ python
def test_f(bu: torch.Tensor) -> None:
    print(bu)
~~~

### Most used functions
1. `test` and `call` from agent
2. `step` from the env and sim
3. `step` from HAC.py
4. `calculate_grads` from HAC.py
5. `apply_grads` from model.py 

Therefore, those functions will be refactored in this order.

### Agent
Each input is converted to numpy, then to a tensor and finally ran through the network. This returns a tensor which gets moved to cpu and converted back to numpy. This is then when storing experiences and when controlling the simulator (in the `step` function). Unfortunately, this cannot be simplified because arrays and tensors are referenced by memory.

### `step` from HAC
This leads to common.py because of unpack_obs. Profiling was done using:
~~~ python
a = np.arange(22)
a = np.expand_dims(a, axis=0)

s = time.time()
grip_pos, object_pos, object_rel_pos, gripper_state, object_velp, object_velr, grip_velp, grip_velr = np.split(a, [3,6,9,10,13,16,19])
print(time.time() - s)

s = time.time()
idx = 0
grip_pos = a[:, idx:idx + 3]
idx += 3
object_pos = a[:, idx:idx + 3]
idx += 3
object_rel_pos = a[:, idx:idx + 3]
idx += 3
gripper_state = a[:, idx:idx + 1]
idx += 1
object_velp = a[:, idx:idx + 3]
idx += 3
object_velr = a[:, idx:idx + 3]
idx += 3
grip_velp = a[:, idx:idx + 3]
idx += 3
grip_velr = a[:, idx:idx + 3]
idx += 3
print(time.time() - s)

####### Results:
2.4318695068359375e-05
4.291534423828125e-06
~~~
Old version seems better.

# Training in the background progress
![Gif](/assets/Reduced-workspace-results/run1.gif)

![Accuracy](/assets/Reduced-workspace-results/accuracy_s.png)
![Actor loss](/assets/Reduced-workspace-results/loss_actor_s.png)
![Critic loss](/assets/Reduced-workspace-results/loss_critic_s.png)
<!-- ![Low level accuracy](/assets/Benefits-of-Normalization/0_accurac.png)
![Low level actor loss](/assets/Benefits-of-Normalization/0_loss_actor.png)
![Low level critic loss](/assets/Benefits-of-Normalization/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Benefits-of-Normalization/1_accuracy.png)
![High level actor loss](/assets/Benefits-of-Normalization/1_loss_actor.png)
![High level critic loss](/assets/Benefits-of-Normalization/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png) -->