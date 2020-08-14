---
layout: post
title:  "Recordings"
date:   2020-08-13 08:25:00 +0100
---
# Generating records
In order to check yesterday's training, the script for generating records has been run. However, all movements in the videos generated looked random and the blue markings (low level target) were not moving. The reason behind it was the 1000 episodes ran for the initialization of the normalizer. In order to overcome this, the option to skip the initialization will be added and static values would be requested instead. The values used will be the ones obtained at the end of the training:

~~~ python
Mean: tensor([ 1.2488e+00,  6.7215e-01,  5.8648e-01,  1.3417e+00,  7.0511e-01,
         4.2495e-01,  9.2777e-02,  3.3163e-02, -1.6046e-01,  3.2886e-02,
         3.2846e-02,  1.6553e-02,  1.9306e-02,  9.1152e-02,  1.9468e-03,
         8.0815e-04, -2.2648e-03,  6.3388e-04,  4.5752e-04,  1.1646e-03,
        -1.9506e-03, -1.6385e-03,  1.1715e-03,  2.5889e-04,  2.9070e-04,
        -2.9732e-02, -9.6862e-02,  7.1217e-02,  1.1615e-02]) 
 Std: tensor([0.1372, 0.2426, 0.1256, 0.1028, 0.1378, 0.0674, 0.1672, 0.2324, 0.1364,
        0.0204, 0.0204, 0.5126, 0.2345, 0.6848, 0.0150, 0.0167, 0.0183, 0.0475,
        0.0375, 0.0501, 0.0153, 0.0171, 0.0167, 0.0078, 0.0078, 0.7241, 0.7104,
        0.6804, 0.0439]) 
 Sum: tensor([ 8.2296e+05,  4.4296e+05,  3.8650e+05,  8.8418e+05,  4.6468e+05,
         2.8005e+05,  6.1142e+04,  2.1855e+04, -1.0574e+05,  2.1672e+04,
         2.1646e+04,  1.0909e+04,  1.2723e+04,  6.0071e+04,  1.2829e+03,
         5.3259e+02, -1.4926e+03,  4.1774e+02,  3.0151e+02,  7.6747e+02,
        -1.2855e+03, -1.0798e+03,  7.7201e+02,  1.7062e+02,  1.9158e+02,
        -1.9594e+04, -6.3834e+04,  4.6933e+04,  7.6542e+03]) 
 Sum_Squares: tensor([1.0401e+06, 3.3651e+05, 2.3707e+05, 1.1932e+06, 3.4016e+05, 1.2200e+05,
        2.4093e+04, 3.6333e+04, 2.9235e+04, 9.8651e+02, 9.8531e+02, 1.7337e+05,
        3.6480e+04, 3.1456e+05, 1.5156e+02, 1.8342e+02, 2.2483e+02, 1.4870e+03,
        9.2717e+02, 1.6567e+03, 1.5712e+02, 1.9355e+02, 1.8517e+02, 4.0016e+01,
        3.9907e+01, 3.4610e+05, 3.3880e+05, 3.0846e+05, 1.3588e+03]) 
 Count: tensor([659018], dtype=torch.int32)
~~~

## Change
Yesterday, one change was mentioned in order to improve training of the high level:
- stop transitions when goal is reached but then check if it's only a 1 length transition. This would remove the transitions when the object is not touched

This modification has been added and a new training has been started. Normalizer has been initialized with the normal 1000 episodes. Moreover, the starting lr of the high level has been increased to 1e-3, since many of the previous experiences have been reduced. The results were not satisfactory.

### Possible improvement
When initializing the normalizer, multiple parallel processes could be used, rather than just one environment.

## Test normalization
In order to check whether normalization is still buggy, two runs will be run. First with normalization enabled, and second with it disabled. Graphs and behaviour will be compared.

After only hundreds of episodes, it becomes obvious that the learning differs when normalization is enabled. The normalization encoding has been disabled for the second run, however, samples were still processed. Therefore, the values obtained at the end of the training will be used as starting point from now on, instead of the usual 1000 episodes initialization. Second training ran for 7k episodes p.p.p.. Here are the values obtained:
~~~ python
Mean: tensor([ 1.4026e+00,  8.1994e-01,  5.8591e-01,  1.3733e+00,  7.6549e-01,
         4.4008e-01, -2.5231e-02, -5.5262e-02, -1.4708e-01,  2.8728e-02,
         2.8737e-02, -1.1265e-02,  1.8171e-02,  5.2092e-03, -2.6046e-04,
        -7.6149e-04, -1.9287e-03, -4.7139e-04,  1.1138e-03,  6.7947e-05,
         7.6746e-04,  9.9249e-04,  9.9618e-04,  2.0966e-04,  1.8413e-04,
         2.2099e-01,  9.0744e-02,  1.0490e-01,  2.4906e-03]) 
 Std: tensor([0.0010, 0.1606, 0.0969, 0.0010, 0.1046, 0.0980, 0.1125, 0.1524, 0.1278,
        0.0211, 0.0211, 0.6198, 0.2231, 0.5176, 0.0112, 0.0139, 0.0145, 0.0585,
        0.0371, 0.0422, 0.0116, 0.0144, 0.0122, 0.0065, 0.0065, 0.5977, 0.6077,
        0.5169, 0.0434]) 
 Sum: tensor([ 3.6142e+06,  2.1129e+06,  1.5098e+06,  3.5389e+06,  1.9726e+06,
         1.1340e+06, -6.5018e+04, -1.4240e+05, -3.7901e+05,  7.4030e+04,
         7.4051e+04, -2.9029e+04,  4.6824e+04,  1.3424e+04, -6.7118e+02,
        -1.9623e+03, -4.9699e+03, -1.2147e+03,  2.8702e+03,  1.7509e+02,
         1.9776e+03,  2.5575e+03,  2.5670e+03,  5.4026e+02,  4.7448e+02,
         5.6947e+05,  2.3384e+05,  2.7031e+05,  6.4179e+03]) 
 Sum_Squares: tensor([5.0242e+06, 1.7989e+06, 9.0881e+05, 4.8498e+06, 1.5382e+06, 5.2379e+05,
        3.4279e+04, 6.7713e+04, 9.7811e+04, 3.2763e+03, 3.2777e+03, 9.9026e+05,
        1.2907e+05, 6.9057e+05, 3.2354e+02, 5.0286e+02, 5.5227e+02, 8.8336e+03,
        3.5494e+03, 4.5967e+03, 3.4656e+02, 5.3450e+02, 3.8397e+02, 1.0935e+02,
        1.0934e+02, 1.0466e+06, 9.7289e+05, 7.1675e+05, 4.8702e+03]) 
 Count: tensor([2576879], dtype=torch.int32) 
~~~

In order to check whether the normalizer is functioning properly with the latest values obtained, a random sample has been normalized and denormalized. Here are the results obtained:

~~~python
After normalization/denormalization:  
 [ 1.3976e+00, 7.4910e-01, 5.3472e-01, 1.3783e+00, 8.4886e-01, 4.2470e-01, 1.1139e-01, 9.9762e-02, -1.1002e-01, 2.5518e-06, 8.7544e-08, -8.8476e-08, 1.3597e-07, 0.0000e+00, -3.4395e-06, 1.0012e-08, 4.6918e-05, 5.0350e-08, -7.7533e-08, 0.0000e+00, 3.4375e-06, 8.7311e-09, -9.0455e-08, 5.1621e-07, 1.9617e-07]


Original sample:  
 [ 1.34194371e+00, 7.49100466e-01, 5.34717228e-01, 1.45333808e+00, 8.48862718e-01, 4.24702091e-01, 1.11394372e-01, 9.97622521e-02, -1.10015137e-01, 2.55108763e-06, -8.67902630e-08, -8.82449685e-08, 1.35761490e-07, 4.08564901e-15, -3.43945358e-06, -1.00249308e-08, 4.69182352e-05, 5.03907166e-08, -7.75241793e-08, -1.15634443e-19, 3.43751849e-06, 8.76712050e-09, -9.04694891e-08, 5.16206905e-07, 1.96171839e-07]

 Delta, rounded to 2 decimal places:  
[ 0.06, -0., 0., -0.08, -0., -0., -0., -0., -0., 0., 0.,-0., 0., -0., -0., 0., -0., -0., -0., 0., -0., -0., 0., 0. -0. ]
~~~

The difference doesn't seem that big, therefore a new train will commence with the above values as initialization values for the normalizer.
<!-- ![Bug found](/assets/Common/bug-stop.png){: .center-image} -->

<!-- ![Low level accuracy](/assets/Normalization-3/0_accuracy.png)
![Low level actor loss](/assets/Normalization-3/0_loss_actor.png)
![Low level critic loss](/assets/Normalization-3/0_loss_critic.png)
![Low level reward](/assets/Normalization-3/0_reward.png)
![High level accuracy](/assets/Normalization-3/1_accuracy.png)
![High level actor loss](/assets/Normalization-3/1_loss_actor.png)
![High level critic loss](/assets/Normalization-3/1_loss_critic.png)
![High level accuracy](/assets/Normalization-3/1_reward.png) -->