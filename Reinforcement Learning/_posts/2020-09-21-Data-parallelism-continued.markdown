---
layout: post
title:  "Data parallelism continued"
date:   2020-09-21 08:30:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Data Parallelism debugging

Problems:
- High level generates "easy" targets in the beggining, meaning that the low level gets rewarded from the first step of the episode
- By hardcoding the target to be far, the low level still fails to learn, moving in the same direction regardless of the state. The loss somehow still goes towards 0, even though there are no good rewards, except the ones added through transitions.

![Bug found](/assets/Common/bug-stop.png){: .center-image}

The optimizer step of the critic was being done after the gradient calculation for the actor. This lead to wrong gradient calculation for the critic because the network is used to predict values in the actor gradient calculation as well. Here is the updated snippet with the correct form:

~~~ python
# train critic
net.crt_opt.zero_grad()
q_v = net.critic(states_v, goals_v, actions_v)
next_act_v = net.tgt_act_net.target_model(
    next_states_v, goals_v)
q_next_v = net.tgt_crt_net.target_model(
    next_states_v, goals_v, next_act_v)
q_next_v[dones_mask] = 0.0
q_ref_v = rewards_v.unsqueeze(dim=-1) + q_next_v * self.gamma
critic_loss_v = F.mse_loss(q_v, q_ref_v.detach())
critic_loss_v.backward()
nn.utils.clip_grad_value_(net.critic.parameters(), 1)
net.crt_opt.step()

# train actor
net.act_opt.zero_grad()
cur_actions_v = net.actor(states_v, goals_v)
actor_loss_v = -net.critic(states_v, goals_v, cur_actions_v)
actor_loss_v = actor_loss_v.mean()
actor_loss_v.backward()
nn.utils.clip_grad_value_(net.actor.parameters(), 1)
net.act_opt.step()		
~~~

## Working on poster

<!-- |  |   |   |   |   |
:-:|:-:|:-:|:-:|:-:|
![Low level accuracy](/assets/Getting-close/0_accuracy.png) | ![Low level actor loss](/assets/Getting-close/0_loss_actor.png) | ![Low level critic loss](/assets/Getting-close/0_loss_critic.png) | ![Low level reward](/assets/Getting-close/0_reward.png)
![High level accuracy](/assets/Getting-close/1_accuracy.png) | ![High level actor loss](/assets/Getting-close/1_loss_actor.png) | ![High level critic loss](/assets/Getting-close/1_loss_critic.png) | ![High level accuracy](/assets/Getting-close/1_reward.png)

![Gif](/assets/Getting-close/run0.gif) -->


<!-- ![Accuracy](/assets/Reduced-workspace-results/accuracy.png)
![Actor loss](/assets/Reduced-workspace-results/loss_actor.png)
![Critic loss](/assets/Reduced-workspace-results/loss_critic.png)

![Gif](/assets/Reduced-workspace-results/run0.gif) -->