---
layout: post
title:  "Running on the supercomputer"
date:   2020-08-25 08:42:00 +0100
---
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->
# Supercomputer
Finalizing the conversion, the script is now ready to run on the supercomputer. An anaconda environment has already been set, however, there was a conflict: `pybullet` and `gym` required incompatible python versions. In order to get past this, `gym` was installed using anaconda, while the `pybullet` was manually installed using pip: `pip install --target=/home/user/.conda/envs/env-name/lib/python3.7/site-packages/ --no-deps`. Also, other packages had to be manually installed using pip, like tensorboardX and ptan.

The job.sh used:
~~~ bash
#!/bin/bash

#SBATCH --job-name=HRL-test
#SBATCH --partition=gpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=8
#SBATCH --time=0:5:0
#SBATCH --mem=24000M
#SBATCH --gres=gpu:1

# manually load modules required
module add languages/anaconda3/2020.02-tflow-2.2.0

# manually export all env vars
export PYTHONPATH=/mnt/storage/home/ro19365/RL-choose-and-place/Bullet-HRL:/mnt/storage/home/ro19365/RL-choose-and-place/panda-gym-env

# load cuda env
source activate learning

# start the script
/mnt/storage/home/ro19365/.conda/envs/learning/bin/python3 /mnt/storage/home/ro19365/RL-choose-and-place/train_HRL_DDPG.py bc4-test
~~~

While setting up the config for the job, I've realized the replay buffer was set for each HAC agent. This means the experiences created are not sampled together by all processes. Therefore, the benefit of parallel processes would be nullified. Looking at how to fix this, it seems like this PyTorch tutorial helps a bit: [link](http://seba1511.net/tutorials/intermediate/dist_tuto.html#our-own-ring-allreduce). Actually, the easiest way would be to duplicate the normalizer code, but for the replay buffer. This means converting the current list into a tensor and calling `share_memory`.

*Note 1*: saves/ and runs/ need to be appended to the `/home` folder path.

*Note 2*: if time runs out, the process is instantaneously killed without firing the `final` statement in the code. This means I need to specifically control the number of iterations the code runs for.



<!-- ![Low level accuracy](/assets/Benefits-of-Normalization/0_accurac.png)
![Low level actor loss](/assets/Benefits-of-Normalization/0_loss_actor.png)
![Low level critic loss](/assets/Benefits-of-Normalization/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Benefits-of-Normalization/1_accuracy.png)
![High level actor loss](/assets/Benefits-of-Normalization/1_loss_actor.png)
![High level critic loss](/assets/Benefits-of-Normalization/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png) -->