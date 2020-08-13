---
layout: post
title:  "Normalization"
date:   2020-07-10 08:30:00 +0100
---
# To Do

Things to do:
- add actions to the normalizer initializer
- pass normalizer to HAC objects
- add call to normalizer in step function
- add call to normalizer in test function
- handle numpy in normalizer, not tensors
- add call to normalizer for updating values

A new function called create_experience was created to help normalize the observations, targets and actions. However, it cannot be used for transitions as well because items inside the tuple are reused for checking whether the goal has been reached.

Both new actions and states are obtained when the `env.step` is called during the low level. That's where the `recompute` function of the normalizer will be called as well.

# Starting to train

After fixing all the obvious bugs, the training started, however after 1.2k episodes it was clear that the high level was not learning too well. Moreover, the low level was very jittery in its movements.

![Bug found](/assets/Common/bug-stop.png){: .center-image}

The `test_env.action_space.sample()` returns values between -1 and 1 for the action, however, the actions are bound between -0.05 and 0.05. In order to fix this, the actions will be sampled based on action boundaries set in the constructor (just like in the agent).