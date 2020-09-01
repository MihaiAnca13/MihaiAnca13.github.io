---
layout: post
title:  "Adjustments"
date:   2020-08-26 08:36:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Things to change
1. update saves/ and runs/ folder path
2. share replay memory between processes
3. use number of iterations/time to stop process before requested time runs out
4. figure out why multiple processes are not spawned

## Updating saves/ and runs/
Current code:
~~~ python
save_path = os.path.join("saves", params['ALGNAME'] + "-" + NAME)
logdir = f"runs/{NAME}-{params['ALGNAME']}"
~~~

This needs to be updated to extract the path in which the code runs and prepend it to the current value. Achieved using: `os.getcwd()`. New code:
~~~ python
save_path = os.path.join(os.getcwd(), "saves", params['ALGNAME'] + "-" + NAME)
logdir = os.path.join(os.getcwd(), "runs", f"{NAME}-{params['ALGNAME']}")
~~~

## Share replay memory between processes
This was achieved by creating a tensor for each part of an Experience (state, action, next_state, ...). Then, using torch's multiprocess package, the tensors were shared using `share_memory_()`. Also, a `lock` was used to make sure no values were added while reading was ongoing in another process. In order to test if it's working, the count was printed each time it got incremented. This resulted in two different counts being displayed: high level and low level buffers.

## 'Time running out' check
The `ep_nr` extracted from the train_queue will be used. This will not represent a very precise estimate of how many iterations have been done, however, it would suffice for now.

## Process not starting
This was solved by changing the following line:
~~~ python
os.environ['OMP_NUM_THREADS'] = "1" # change 1 to number of processes + 1 (number of HAC agent + 1 thread for the test environment)
~~~

*Note*: CPU in BlueCrystal means cpu core, therefore a total of 25 were requested: 1 per environment spawned (16), 1 per simulator thread (8) and finally 1 for the main script.

# Develop High Only
Now that the training is on the job list, it's time to reduce the complexity even more. Since the low level learns quite easily, the problem can be reduced to a simple DDPG training of the high level. This is what develop-high-only branch will be about.



<!-- ![Low level accuracy](/assets/Benefits-of-Normalization/0_accurac.png)
![Low level actor loss](/assets/Benefits-of-Normalization/0_loss_actor.png)
![Low level critic loss](/assets/Benefits-of-Normalization/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Benefits-of-Normalization/1_accuracy.png)
![High level actor loss](/assets/Benefits-of-Normalization/1_loss_actor.png)
![High level critic loss](/assets/Benefits-of-Normalization/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png) -->