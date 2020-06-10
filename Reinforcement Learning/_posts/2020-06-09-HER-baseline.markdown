---
layout: post
title:  "Understanding Hindsight Experience Replay OpenAI's implementation"
date:   2020-06-09 08:01:00 +0100
---
# Introduction
Hindisht Experience Replay ([HER](https://arxiv.org/pdf/1707.01495.pdf)) implemented at [OpenAI](https://github.com/openai/baselines/tree/master/baselines/her)  
At the time of this writing, the code was written in tensorflow 1.14, and it's not yet converted to tf2. The idea is to convert the code to PyTorch, a library I am more familiar with. I expect that in the process, I will gain a deeper understading of the algorithm.

There are other implementations of HER on github, however, OpenAI's implementation succesfully converges on a solution for the Pick and Place environment. For example, I've found [this](https://github.com/nikhilbarhate99/Hierarchical-Actor-Critic-HAC-PyTorch/tree/master), applied on the continious mountain car environment.

# HER vs Hierarchical HER
The file her.py has a couple of parameters:  
- env: FetchPickAndPlace-v1
- total_timesteps: 500000
- seed: 0
- policy\_save\_interval: 0 (only the best and last policies would be saved)
- replay_strategy: future (activates her)
- clip_return: 1 (don't know what it means just yet)
- demo_file: don't know what it means just yet

The file config.py contains all the default parameters. I will start by looking at the DDPG structure. From the config, the network has 3 dense layers, with 256 hidden neurons each. Each layer is followed by a ReLu activation. From the actor_critic.py file, we get the input size, which is the concatenation of the observation and goal. Moreover, the actor network is wrapped in a tanh activation. The number of output neurons is equal to the action space. On the other hand, the critic has the observation, goals and actions as input. The output is one neuron which would predict the Q value. There is no activation.

In PyTorch, this would look like this:

~~~ python
actor = nn.Sequential(
    # state + goal
    nn.Linear(obs_size + obs_size, HID_SIZE),
    nn.ReLU(),
    nn.Linear(HID_SIZE, HID_SIZE),
    nn.ReLU(),
    nn.Linear(HID_SIZE, HID_SIZE),
    nn.ReLU(),
    nn.Linear(HID_SIZE, act_size),
    nn.Tanh()
)

critic = nn.Sequential(
    nn.Linear(obs_size + obs_size + act_size, HID_SIZE),
    nn.ReLU(),
    nn.Linear(HID_SIZE, HID_SIZE),
    nn.ReLU(),
    nn.Linear(HID_SIZE, HID_SIZE),
    nn.ReLU(),
    nn.Linear(HID_SIZE, 1),
)
~~~

Lines 376 and 377 in ddpg.py show how the loss is calculated, however, I'll come back to that later on. I've noticed the output of the critic network is clipped to positive values only. I don't understand exactly how the values are chosen, but I'll return to this. 

In the paper there is a variable H which dictates the number of steps each level takes before receiving a new goal. In the Open AI's implementation I can only find T, which dictates the size of trajectories. There is no passing of goals to lower levels. This is because OpenAI implements Highsight Experience Replay, while I tried to use Hierarchical HER.

I will revert the changes and try to solve the Pick and Place environment with my current setup, but with similar hyperparameters as OpenAI.

# Modifying the original code for Pick and Place

I've started by adding one extra layer to both networks. Also, the critic network now has the observation, goal and action as the input to the first layer, rather than action being concatenated with the output of the first layer.

Next thing I need to figure out is how the state/action boundaries/offsets change. Each obsercation is a dictionary containing the current state, the desired goal and the achieved goal. I am going to use this as the state and goal. The state information has changed, therefore a new unpack_obs function must be defined.

Here's how the high/low levels work:

- You have a state and a goal 
- The state is not common for both levels 
- The high level state contains all the info 
- The low level state only contains the gripper position and gripper state 
- The high level produces a gripper position and state that the low level must achieve 

After checking the maximum state achievable by the robot arm, I've realized the boundaries are already correctly set. I have to add possibility of completly random action. Here, the noise added to position and gripper must be drawn from separate distributions. Each axes takes values between different boundaries, therefore the noise amplitude should differ. This is achived by creating a separate distribution for each axis and gripper using the boundary values.

I've started the training with the following hyperparameters:
~~~ python
GAMMA = 0.95
BATCH_SIZE = 256
LEARNING_RATE = 1e-4
REPLAY_SIZE = 1_000_000
REPLAY_INITIAL = 1_000
H = 20
TEST_ITERS = 100
N_ITERS = 50
RANDOM_EPS = 0.3
NOISE_EPS = 0.2
~~~

Training seems to be failing. However, I've noticed that the end-effector would go outside the state boundaries. I will investigate this by hacking into the environment and displaying the intrinsic target generated by the high level model. High level actor and critic loss seems to be frozen at 20 and 0 respectively. This must be investigated as well.


