---
layout: post
title:  "Normalization results"
date:   2020-08-17 08:25:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Results
Training with normalization enabled did not go very far because the high level model overfitted after only 400 steps. Going forward, normalization will therefore be disabled.

The other results obtained with no normalization showed signed of improvement, however, the grasping was still not learned. The OpenAI baseline for HER will be run locally to check whether fewer parallel processes has an impact on learning other than time efficiency. This could show that our algorithm could successfully learn if only multiple cores would be available. If that's the case, the transition to PyBullet physics engine would be the next step.

The baseline algorithm cannot be run on my local machine due to limited amount of physical cores. In one paper, they've used 19 cores, in another 8, which translates to the following numbers:  
HER original - 320k episodes - 8 physical cores  
OpenAI baseline: 1.8mil episodes - 19 physical cores  
Mine - 120k episodes - 1 physical core

The problem is that their graphs show steady increase in accuracy. Our current algorithm doesn't produce that. Plus, all papers use normalization, which in our case did not help the training.

## Next steps?
What would need to be done to convert?
1. create "fetch pick and place" in PyBullet
2. adjust current code to run with new environment
3. figure out normalization
4. possible need to run code with mpirun - used by OpenAI for parallel processing over multiple cores

### Risks
Running out of time is the primary concern. Other risk is that the algorithm cannot simply learn the task at hand without demonstrations. However, jumping ahead at using demonstrations could prove catastrophic if the currently running algorithm has bugs.

## Notes from re-reading HAC
The biggest difference was in breaking out of loop when goal achieved - this change was made because jittery movements were used to cover a wider area with the end-effector.


<!-- ![Low level accuracy](/assets/Benefits-of-Normalization/0_accurac.png)
![Low level actor loss](/assets/Benefits-of-Normalization/0_loss_actor.png)
![Low level critic loss](/assets/Benefits-of-Normalization/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Benefits-of-Normalization/1_accuracy.png)
![High level actor loss](/assets/Benefits-of-Normalization/1_loss_actor.png)
![High level critic loss](/assets/Benefits-of-Normalization/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png) -->