---
layout: post
title:  "Converting to PyBullet"
date:   2020-08-19 08:25:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# PyBullet package
Going through the documentation, getting started guide and local files, I have found that there already is a Franka Panda package for PyBullet. This is located at: `/site-packages/pybullet_data/franka_panda`. This provides the urdf file and collision meshes required to simulate the robot. Moreover, there already is a sample code that spawns in an environment with a panda arm and a tray filled with objects. Next step is to isolate that environment and change it so it only has one object that is spawned randomly within the tray.

![Sim](/assets/Converting-to-PyBullet/sim_1.png)

There is the gym file for "pick_and_place" environment. There is the "kukaGymEnv" environment written by pyBullet as an example. Now I need to create mine, but also change the `model.py` file where it calls the mujoco simulator directly. Also, I must make the movements behave the same way as with the mocap from mujoco. This will be achieved using inverse kinematics. When I've created the 9 objects environment, the class would inherit form the gym's `robot_env.RobotEnv` class. Unfortunately, I am unable to do so now, because all functions are based on the mujoco sim. Therefore, everything must be re-implemented.

Another thing to keep in mind is the workspace dimensions. This means that all boundaries will have to be updated to match the new environment.

Actually, there must be 2 classes created: one for the simulator and one for the environment.

## Simulator Class
Name: FetchBulletSim

Functions:
- init:
        - spawn objects + robot
        - save state
- randomize box position:
        - move box to random position in the air or on the table, relatively close to the end effector
- step:
        - set gripper
        - move end effector
- reset:
        - restore randomly chosen state

## Environment Class
Name: FetchBulletEnv

Functions:
- step
- reset
- 

!Note: Check gym utils `seeding`

The simulation starts with the arm in a corner, however, it would be better if the end effector would start in the middle of the tray. Here are the steps taken to get a new starting position:
- manually move the end effector roughly in the middle
- print position
- use IK to calculate joint positions
- pre-set the orientation
- use new joint positions
- * if the IK returns weird configuration, use the old joint positions to initialize robot and then calculate IK

Here are the new joint pos:
~~~ python
jointPositions = [1.076, 0.060, 0.506, -2.020, -0.034, 2.076, 2.384, 0.03, 0.03]
~~~

Next steps are adding support for actions when calling `step` and adding a rendering option. The latter will probably have to be done with some other tools since the `DIRECT` mode is the main one used for speed during training. This means that the normal view-port will not be usable.

The simulator and library work like a server and API. However, I cannot disconnect from direct mode and into GUI, because the states saved in memory are getting erased on disconnect.

Being said that, can the simulator have multiple processes in parallel? It seems to be working if two scripts are started, however, how am I supposed to get two instanced running under the same process? Here's the answer: `from pybullet_utils import bullet_client as bc` ([source](https://github.com/bulletphysics/bullet3/blob/master/examples/pybullet/gym/pybullet_utils/examples/multipleScenes.py)).

<!-- ![Low level accuracy](/assets/Benefits-of-Normalization/0_accurac.png)
![Low level actor loss](/assets/Benefits-of-Normalization/0_loss_actor.png)
![Low level critic loss](/assets/Benefits-of-Normalization/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Benefits-of-Normalization/1_accuracy.png)
![High level actor loss](/assets/Benefits-of-Normalization/1_loss_actor.png)
![High level critic loss](/assets/Benefits-of-Normalization/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png) -->