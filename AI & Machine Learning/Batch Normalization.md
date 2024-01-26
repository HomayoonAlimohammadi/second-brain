#ai #batch_norm #neural_networks

## related:
[[Neural Network Optimization]]
## references:
https://towardsdatascience.com/batch-norm-explained-visually-how-it-works-and-why-neural-networks-need-it-b18919692739
https://towardsdatascience.com/batch-norm-explained-visually-why-does-it-work-90b98bcc58a0

![[batch_norm 1.webp]]
- Calculate mean and std dev for previous layer activations 
	- use these mean and std dev to normalize the activations
	- scale and drift the activations (with learnable parameters)
	- `This step is the huge innovation introduced by Batch Norm that gives it its power. Unlike the input layer, which requires all normalized values to have zero mean and unit variance, Batch Norm allows its values to be shifted (to a different mean) and scaled (to a different variance). It does this by multiplying the normalized values by a factor, gamma, and adding to it a factor, beta. Note that this is an element-wise multiply, not a matrix multiply. What makes this innovation ingenious is that these factors are not hyperparameters (ie. constants provided by the model designer) but are trainable parameters that are learned by the network. In other words, each Batch Norm layer is able to optimally find the best factors for itself, and can thus shift and scale the normalized values to get the best predictions.`
- update the EMA for mean and std in order to later use in inference


# Why batch norm works:
## 1- Internal Covariate shift:
Batch Norm helps to stabilize these shifting distributions from one iteration to the next, and thus speeds up training
![[batch_norm_cov_shift.webp]]

![[Batch Normalization-Accelerating Deep Network Training by Reducing Internal Covariate Shift.pdf]]

## 2- Loss and Gradient Smoothing
Batch Norm significantly smoothens the loss landscape

![[batch_norm_loss_1.webp]]
![[batch_norm_loss_2.webp]]


## When not to use:
Batch Norm doesn’t work well with **smaller batch sizes**. That results in too much noise in the mean and variance of each mini-batch.

Batch Norm is not used with **recurrent networks**. Activations after each timestep have different distributions, making it impractical to apply Batch Norm to it.

![[How Does Batch Normalization Help Optimization.pdf]]