---
layout: post
title:  "Contradiction"
date:   2020-07-03 08:30:00 +0100
---
# Discussion
While trying to find possible problems with the current implementation, the following points were noted down:
- transition gamma - should it only be 0 at the end?
- double check observation unpacking
- plot average distance between end-effector and object/episode
- display gripper state by adding a secondary blue ball in the environment
- HER: transition or experience, not both

## 1. Transition gamma
Currently only transitions longer than one sample are considered. The last entry in the transition has their gamma and reward set to 0. Then, for each entry, the reward is recalculated if the goal (last state in the transition) has been reached.

## 2. Observation unpacking
During training, multiple test renderings show the arm missing the object but aiming very close. This could imply that there might be an error in the unpacking of the observations.

## 3. Average distance
Through training, the model should learn that spending time connected to the object brings better results. This should be easily observed by plotting the average distance between the end-effector and the object. It would be another metric to help debugging. This issue is strongly linked to second point.

## 4. Gripper state
The accuracy of the low level model is quite low despite the performance observed through test renderings. The hypothesis is that the gripper state is not matching the goal gripper state, however the position is more easily achieved. In order to test this theory, the desired gripper state can be shown by adding a secondary blue ball to the environment and positioning them in such a way to display where the gripper's fingers should be.

## 5. HER
Let's analyse the case where the robot successfully achieves the goal of grasping the object from the table and moving it in the air. Notations:
- P1 = initial gripper position and state at the beginning of the episode
- P2 = position of the object on the table with the gripper open
- P3 = position of the object on the table with the gripper closed
- P4 = goal position in the air with the gripper grasping (the object)

During this episode, the high level would take 5 actions. Each action would then add an experience and a transition to the replay buffer. Let's have a look at what it looks like after each loop, assuming the low level is perfectly trained:

### Loop 1
Goal hasn't been achieved. The following experience and transition are created:
~~~ python
# replay buffer:
exp = Experience(state=P1, action=P2, next_state=P2, reward=-1, done=False, gamma=0.98, goal=P4)
# transitions list:
transition = Experience(state=P1, action=P2, next_state=P2, reward=-1, done=False, gamma=0.98, goal=None)
~~~

### Loop 2
Goal still hasn't been achieved. Experiences and transitions created:
~~~ python
exp = Experience(state=P2, action=P3, next_state=P3, reward=-1, done=False, gamma=0.98, goal=P4)
transition = Experience(state=P2, action=P3, next_state=P3, reward=-1, done=False, gamma=0.98, goal=None)
~~~

### Loop 3
Goal still has been achieved with next state. Experiences and transitions created:
~~~ python
exp = Experience(state=P3, action=P4, next_state=P4, reward=0, done=False, gamma=0, goal=P4)
transition = Experience(state=P3, action=P4, next_state=P4, reward=-1, done=False, gamma=0.98, goal=None)
~~~

### Loop 4
Goal achieved. Experiences and transitions created:
~~~ python
exp = Experience(state=P4, action=P4, next_state=P4, reward=0, done=False, gamma=0, goal=P4)
transition = Experience(state=P4, action=P4, next_state=P4, reward=-1, done=False, gamma=0.98, goal=None)
~~~

### Loop 5
Goal achieved and episode done. Experiences and transitions created:
~~~ python
exp = Experience(state=P4, action=P4, next_state=P4, reward=0, done=True, gamma=0, goal=P4)
transition = Experience(state=P4, action=P4, next_state=P4, reward=-1, done=True, gamma=0, goal=None)
~~~

### Parsing transitions:
Setting goal to last state (P4) and recalculating reward:
~~~ python
transition1 = Experience(state=P1, action=P2, next_state=P2, reward=-1, done=False, gamma=0.98, goal=P4)
transition2 = Experience(state=P2, action=P3, next_state=P3, reward=-1, done=False, gamma=0.98, goal=P4)
transition3 = Experience(state=P3, action=P4, next_state=P4, reward=0, done=False, gamma=0.98, goal=P4)
transition4 = Experience(state=P4, action=P4, next_state=P4, reward=0, done=False, gamma=0.98, goal=P4)
transition5 = Experience(state=P4, action=P4, next_state=P4, reward=0, done=True, gamma=0, goal=P4)
~~~

### Final replay buffer:
~~~ python
exp1        = Experience(state=P1, action=P2, next_state=P2, reward=-1, done=False, gamma=0.98, goal=P4)
transition1 = Experience(state=P1, action=P2, next_state=P2, reward=-1, done=False, gamma=0.98, goal=P4)

exp2        = Experience(state=P2, action=P3, next_state=P3, reward=-1, done=False, gamma=0.98, goal=P4)
transition2 = Experience(state=P2, action=P3, next_state=P3, reward=-1, done=False, gamma=0.98, goal=P4)

exp3        = Experience(state=P3, action=P4, next_state=P4, reward=0,  done=False, gamma=0,    goal=P4)
transition3 = Experience(state=P3, action=P4, next_state=P4, reward=0,  done=False, gamma=0.98, goal=P4)

exp4        = Experience(state=P4, action=P4, next_state=P4, reward=0,  done=False, gamma=0,    goal=P4)
transition4 = Experience(state=P4, action=P4, next_state=P4, reward=0,  done=False, gamma=0.98, goal=P4)

exp5        = Experience(state=P4, action=P4, next_state=P4, reward=0,  done=True,  gamma=0,    goal=P4)
transition5 = Experience(state=P4, action=P4, next_state=P4, reward=0,  done=True,  gamma=0,    goal=P4)
~~~

## Thoughts
There are 3 duplicates and 2 entries that even though they have the same state, action and next_state, their gamma values are different. In this case, it won't make a difference because the q value of this combination would be 0.

What if negative experiences are generated when the goal is not achieved, but positive transitions are generated when the goal is achieved? There would not be enough good transitions.

What if positive transitions would be generated when the goal is not achieved, and experiences generated when the goal is achieved? There is no need to reward with 0 actions in the beginning of the transition, since q values would propagate backwards multiplied by gamma. Does this mean that a larger gamma would benefit transitions more? Gamma of 0.98 is high already, but will run a test with 0.99.

## Unsuccessful example
Introducing P5 = position and state of gripper at the end of episode. Let's assume the robot it's grasping the object and that it passes through P4.  
Final replay buffer, after one episode, would look like this:
~~~ python
exp1        = Experience(state=P1, action=P2, next_state=P2, reward=-1, done=False, gamma=0.98, goal=P4)
transition1 = Experience(state=P1, action=P2, next_state=P2, reward=-1, done=False, gamma=0.98, goal=P5)

exp2        = Experience(state=P2, action=P3, next_state=P3, reward=-1, done=False, gamma=0.98, goal=P4)
transition2 = Experience(state=P2, action=P3, next_state=P3, reward=-1, done=False, gamma=0.98, goal=P5)

exp3        = Experience(state=P3, action=P4, next_state=P4, reward=0,  done=False, gamma=0,    goal=P4)
transition3 = Experience(state=P3, action=P4, next_state=P4, reward=-1, done=False, gamma=0.98, goal=P5)

exp4        = Experience(state=P4, action=P5, next_state=P5, reward=-1, done=False, gamma=0.98, goal=P4)
transition4 = Experience(state=P4, action=P5, next_state=P5, reward=0,  done=False, gamma=0.98, goal=P5)

exp5        = Experience(state=P5, action=P5, next_state=P5, reward=-1, done=True,  gamma=0,    goal=P4)
transition5 = Experience(state=P5, action=P5, next_state=P5, reward=0,  done=True,  gamma=0,    goal=P5)
~~~

# Next steps
Since number 1 and 5 on the initial list have been addressed and there are no findings, I will continue with number 4.

<img src="/assets/Contradiction/gripper_state.png" alt="gripper state" width="500"/>

## Result
By plotting the desired position of the low level while rendering, it became obvious why the low level accuracy doesn't reflect the successful behaviour: when the object is grasped, the gripper cannot be fully closed, therefore the goal cannot be achieved. A check should be added to permit better rewards for the low level. The following code checks for this case:
~~~ python
ignore = False
# if target is grasping
if target[3] < 0.02:
    for i in range(self.env.sim.data.ncon):
        geom1_id = self.env.sim.data.contact[i].geom1
        body1_id = self.env.sim.model.geom_bodyid[geom1_id]
        name1 = self.env.sim.model.body_id2name(body1_id)

        geom2_id = self.env.sim.data.contact[i].geom2
        body2_id = self.env.sim.model.geom_bodyid[geom2_id]
        name2 = self.env.sim.model.body_id2name(body2_id)

        # if object is between fingers
        if ('object' in name1 and 'finger' in name2) or ('finger' in name1 and 'object' in name2):
            ignore = True
            break

if ignore:
    ignore_threshold = self.low_thresholds
    ignore_threshold[-1] = 1
    goal_achieved = goal_distance(low_level_next_state, target, ignore_threshold)
else:
    goal_achieved = goal_distance(low_level_next_state, target, self.low_thresholds)
~~~

In only 200 steps, the low level achieves 80% accuracy! However, the test renderings show that the gripper is always fully opened. I've just witnessed the target being above the object with the gripper closed. However, even though the position of the gripper was correct, it wasn't grasping. It could be that both fingers should be checked for collision with the object. 
**Massive** bug: the state generated by the high level could have negative values for the gripper state. Another annoyance was this piece of code:
~~~ python
ignore_threshold = self.low_thresholds
ignore_threshold[-1] = 1
~~~

This `self.low_thresholds` variable is a numpy array, therefore, when copied, it's copied by reference. When setting gripper state threshold to 1, it was also overwriting `self.low_thresholds[-1]` to 1. After fixing the bugs, 80% accuracy is achieved after 500 steps, however it struggles to improve. This could be problematic.