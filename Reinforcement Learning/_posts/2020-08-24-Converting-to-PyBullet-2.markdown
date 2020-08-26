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
- update rendering calls - move finger markers (this needs to be checked because the simulator is stepping only at each action. Therefore, should the markers be moved before the action?)
- update workspace constraints

After the script is running on local machine, pybullet must be installed on the super computer and the job.sh bash script updated.

Detecting collisions in pybullet is very elegant:
~~~ python
def detect_gripper_collision(self):
    aabbMin, aabbMax = self.bullet_client.getAABB(self.panda, 9)
    # drawing collision line?
    # f = [aabbMax[0], aabbMin[1], aabbMin[2]]
    # t = [aabbMax[0], aabbMax[1], aabbMin[2]]
    # self.bullet_client.addUserDebugLine(f, t, [1, 1, 1])
    body_link_ids = self.bullet_client.getOverlappingObjects(aabbMin, aabbMax)
    body_ids = [x[0] for x in body_link_ids]
    if self.cubeId in body_ids:
        return True
    return False
~~~

## Calculating boundaries:
In this simulation, the y axis is controlling the height. This is how it was set for the kuka arm in the example provided and I am not sure how to change it to z axis. Therefore, here are the values obtained by moving the arm around and printing the position of the gripper:
~~~ python
state boundaries:
0 - left/right, 1 - top/down, 2 - forward/backward
min: [-0.221, 0.027, -0.465]
max: [0.185, 0.35, -0.730]
~~~

# PyBullet bug
When the cube is initialized in the air and the time_step is set too low (e.g.: 60), the cube will fall through the tray. This will happen even when not rendering and the time_step is set to the default (240). In order to address this issue, the following code was used to teleport the cube back on the tray:

![Bug found](/assets/Common/bug-stop.png){: .center-image}

~~~ python
c = self.bullet_client.getBasePositionAndOrientation(self.cubeId)[0]
if c[1] < 0.034:
    c = list(c)
    c[1] = 0.034
    orn = self.bullet_client.getQuaternionFromEuler([math.pi / 2., 0., 0.])
    self.bullet_client.resetBasePositionAndOrientation(self.cubeId, c, orn)
~~~
<!-- ![Low level accuracy](/assets/Benefits-of-Normalization/0_accurac.png)
![Low level actor loss](/assets/Benefits-of-Normalization/0_loss_actor.png)
![Low level critic loss](/assets/Benefits-of-Normalization/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Benefits-of-Normalization/1_accuracy.png)
![High level actor loss](/assets/Benefits-of-Normalization/1_loss_actor.png)
![High level critic loss](/assets/Benefits-of-Normalization/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png) -->