#evolutionary #evolugionary_algorithms #classification #deep_learning #architecture_search #image_classification #ai #machine_learning

## Highlights:
- **Aging Evolution** can only improve a population through the inheritance of architectures that re-train well. (In contrast, NAE can improve a population by accumulating architectures/models that were lucky when they trained the first time). That is, AE is forced to pay attention to architectures rather than models. In other words, the addition of aging involves introducing additional information to the evolutionary process: architectures should re-train well. This additional information prevents overfitting to the training noise, which makes it a form of regularization in the broader mathematical sense.
- The mutation probabilities could be **learned** to improve speed
- We proposed aging evolution, a variant of tournament selection by which genotypes die according to their age, favoring the young. This improved upon standard tournament selection while still allowing for efficiency at scale through asynchronous population updating.
## Introduction and Background:
The paper begins by discussing the motivation for using architecture search to automatically discover image classifiers, emphasizing the inefficiency of manually designed architectures. The authors propose a variant of the tournament selection evolutionary algorithm called "aging evolution," which biases the selection towards younger genotypes. They also implement mutations allowing evolution in the NASNet search space, enabling a direct comparison with reinforcement learning (RL) methods.

## Methodology:
The authors describe their evolutionary algorithm, highlighting the use of the NASNet search space and the introduction of simple mutations for evolving neural network architectures. The paper details the structure of the search space and the process of architecture discovery through mutations. They also explain the experimental setup, including model training and evaluation on datasets like CIFAR-10 and ImageNet.

## **Results and Comparisons**:
The paper presents a comparative study of their evolutionary approach with RL and random search baselines, focusing on the speed and quality of the search. They demonstrate that their method achieves similar or better results compared to RL, especially in early stages or under resource constraints. The evolution-based AmoebaNet-A model is introduced, which shows competitive performance with other top image classifiers.

## Discussion and Future Work:
The authors speculate on the benefits of aging evolution, suggesting it may navigate training noise better than non-aging methods. They propose future research directions, including extending their comparative studies to more search spaces and tasks, and exploring the potential of aging evolution in larger search spaces.

## Conclusion:
The paper concludes by summarizing the key contributions, including the proposal of aging evolution, the comparative study with other architecture search methods, and the development of the AmoebaNet-A model, which surpasses hand-designed classifiers and sets a new accuracy benchmark on ImageNet.

## Code:
https://colab.research.google.com/github/google-research/google-research/blob/master/evolution/regularized_evolution_algorithm/regularized_evolution.ipynb

## File:
![[Regularized Evolution for Image Classifier Architecture Search.pdf]]