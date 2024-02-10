#evolugionary_algorithms #neural_networks #architecture_search #architecture_optimization #optimization 

### Reference:
https://arxiv.org/pdf/1808.07233.pdf

#### Continuous vs Discrete:

* Operates in a continuous space unlike traditional methods that perform architecture search in a discrete space
* Utilizes gradient based optimization in order to achieve higher quality models

#### Key Components:
- Encoder:
	- Maps neural network architectures into a continuous space

* Performance Predictor:
	* Takes the continuous representation of a network as input and predicts its performance (e.g. accuracy)
* Decoder:
	* Maps the continuous representation back to a network architecture

![[Screenshot 2024-02-07 at 7.54.45 in the morning.png]]

#### Encoder
- An LSTM model encodes the architecture descriptions into a continuous space
- Each architecture is represented as a sequence of tokens, and the LSTM’s hidden state serve as the continuous representation

#### Performance Predictor
* Simple feed-forward neural network that takes the encoded representation as input and outputs a performance prediction (e.g. expected accuracy)
* This network is trained to minimize the difference between predicted and actual performance of architectures
#### Decoder
* This network is trained to minimize the difference between predicted and actual performance of architectures
* Incorporates an attention mechanism to improve the accuracy of the decoding process

#### Optimization Process
* Incorporates an attention mechanism to improve the accuracy of the decoding process
* Targets improving the performance predictor’s output for a given architecture
![[Screenshot 2024-02-07 at 8.06.21 in the morning.png]]


#### Integration and Training
* The encoder, performance predictor and the decoder are jointly trained with the architecture performance data
* The trained model is used to generate new architectures by encoding, optimizing and decoding

![[Screenshot 2024-02-07 at 8.05.14 in the morning.png]]

#### Paper

![[Neural Architecture Optimization.pdf]]