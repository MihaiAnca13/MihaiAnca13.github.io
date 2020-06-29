---
layout: post
title:  "New Approach"
date:   2020-06-24 10:50:00 +0100
---
# Initial state
Training failed in all attempts, therefore, the idea of having two different initial states will be investigated. One initial state would be unchanged, meaning the robot would be above the table, the box on the table. The other one would be with the robot initially grasping the object. There are multiple ways of implementing this:
- moving the object at the initial gripper position, setting gripper to squeeze
- moving object to random position in proximity of initial gripper position, move gripper on top of object, then squeeze

The second approach would have the advantage of exploring a larger number of initial states, leading to potentially better training.

In order to create this, a script that manually moved the robot was created:
~~~ python
init = env.sim.data.site_xpos[2].copy() # get initial position of gripper

for i in range(10):
    # open
    env.step(np.array([0., 0., 0., 1]))
    env.sim.step()
    env.step(a)
    env.render()

for i in range(10):
    # move to object
    b = env.sim.data.site_xpos[3]
    b[-1] += 0.03
    env.sim.data.mocap_pos[:] = b
    env.sim.step()
    env.step(a)
    env.render()

for i in range(10):
    # close
    env.step(np.array([0., 0., 0., -1.]))
    env.sim.step()
    env.render()

for i in range(10):
    # move to initial pos
    env.sim.data.mocap_pos[:] = init
    env.sim.step()
    env.step(np.array([0., 0., 0., -1.]))
    env.render()

print(env.sim.get_state())
~~~

The data extracted at the end of the script was hardcoded in the environment setup function. The simulator chooses between the initial and the new state with a chance of 50%.

## Possible improvements
One of the processes gets ahead of the others. This is because the first one to reach an episode that gets tested will get to run for two more loops before being stopped again. The other processes have to wait until the test runs before their calculated grads get integrated. This could be soved by testing when the last process reaches the tested episode number. However, implementing this would make the code harder to understand and doesn't gain any benefit in terms of training.

Another minor problem is the debug messages. Every process displays one at the end of an episode, however, the process number is not included. This debug message is displayed using the library ptan, and inlcu