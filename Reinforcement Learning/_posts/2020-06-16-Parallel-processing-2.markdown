---
layout: post
title:  "Parallel processing 2"
date:   2020-06-16 08:09:00 +0100
---
# Implementation
There are two types of actor-critic parallelization:
1. Data parallelism: each process interacts with one or more environments and stores transitions
2. Gradients parallelism: each process calculates gradients on their own training samples

Given that eventually, I will run the training on a super computer, the most effective approach would be to implement the gradients parallelism. Also, the [HER](https://arxiv.org/pdf/1707.01495.pdf) paper used gradients parallelization as well.

 ![Gradients Parallelization](/assets/Parallel-processing/gradientsP.png){: .center-image}
 <div>
 	<em>Gradients Parallelization (Deep Reinforcement Learning Hands-On - Second Edition, Maxim Lapan, 2020)</em>
 </div>{: .center-text}

The HAC agent created needs to be rewritten. This is because, currently, the agent interacts with the environment at each step and writes the experiences in the replay buffer. The paralellization process requires each process to have its own unique memory replay buffer, while sharing the parameters of the network.

The new class would be inheriting the old class, because the \_\_init\_\_ function of the current HAC agent is creating the networks. Most of the class can be reused, except for the train_n_level, which would be replaced by a calculate_grads function. However, this would require all the initial parameters to be reset. A better approach would be to strip the model creation from the HAC class. Therefore, a new class called Models would take care of the initialization of the models and will share the network with all subsequent HAC classes. The HAC class will be modified to accept the Models class as an input and work with multiple environments.

I was unsure wheter it's correct for each process to have it's own replay memory. I found those two papers called [Accelerated Methods for Deep Reinforcement Learning](https://arxiv.org/pdf/1803.02811.pdf) and [Asynchronous Episodic Deep Deterministic PolicyGradient: Towards Continuous Control inComputationally Complex Environments](https://arxiv.org/pdf/1903.00827.pdf). They describe asyncronous architectures in which each process keeps track of its own replay memory, however the total size of samples saved is the same.

I've noticed that before parallelization, the training was being done every time an episode ended. Now, the training would be done once per loop. I'm curious to see if this would have a drastic effect on the results.