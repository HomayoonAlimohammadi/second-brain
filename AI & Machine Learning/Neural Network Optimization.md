#ai #neural_networks #optimization 

1. **Role of Optimizers**: They are crucial in adjusting model parameters to produce the best outputs for the problem being solved. Gradient descent is the most common optimization technique.

2. **Variety of Optimizers**: Deep learning libraries like Pytorch and Keras offer many optimizers based on gradient descent, such as SGD, Adadelta, Adagrad, RMSProp, and Adam. The article aims to provide context and intuition about how each algorithm fits into the broader picture, rather than delving into the mathematical formulas.

3. **Loss Curve and Gradient Descent**: The article describes the loss curve in a 3D representation, showing how gradient descent navigates the loss landscape of a neural network. It emphasizes the computation of gradients and the update of weights based on these gradients and a learning rate factor.

4. **Challenges with Gradient Descent**: The article discusses issues like local minima, saddle points, and ravines that gradient descent faces, and how these challenges affect the optimization process.

5. **Improvements to Gradient Descent**:
	- **Stochastic Gradient Descent (SGD)**: Uses subsets of data for each training iteration, adding randomness to explore the loss landscape.
	- **Momentum**: Adjusts the update amount dynamically, helping to navigate steep slopes and ravines by considering past gradients.
	- **Adaptive Learning Rates**: Algorithms like Adagrad, Adadelta, and RMSProp adjust the learning rate for each parameter based on past gradients.
	- **Schedulers**: Adjust the learning rate based on training progress, independent of model parameters.

1. **Conclusion**: The article sets the stage for a deeper understanding of specific optimization algorithms, explaining the basic techniques and their interrelation

### referrence:
https://towardsdatascience.com/neural-network-optimizers-made-simple-core-algorithms-and-why-they-are-needed-7fd072cd2788