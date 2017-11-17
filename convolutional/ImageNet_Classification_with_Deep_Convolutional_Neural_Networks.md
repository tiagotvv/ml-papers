### Paper

+ **Title**: ImageNet Classification with Deep Convolutional Neural Networks
+ **Authors**: Alex Krizhevsky, Ilya Sutskever, Geoffrey E. Hinton
+ **URL**: [https://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks.pdf](https://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks.pdf)
+ **Year**: 2012
+ **Other information**: Published in Advances in Neural Information Processing Systems (NIPS 2012).
+ **Key**: Krizhevsky2012


#### Goal:
+ Train a deep convolutional neural network to classify 1.2 million images into 1000 different categories.

#### Convolutional Neural Networks:
+ Make strong and correct assumptions about the nature of the images (stationarity, pixel dependencies). 
+ Much fewer connections and parameters: easier to train than fully connected neural networks.

#### Dataset
+ ImageNet: 15 million labeled high-resolution images from 22000 categories. Labeled manually using Amazon Mechanical Turk.
+ ImageNet Large-Scale Visual Recognition Challenge (ILSVRC): subset of ImageNet
    + 1.2 million training images, 50000 validation images, 150000 test images.
    + 1000 categories
+ Variable resolution images:
    + Images downsampled to a fixed resolution of 256 x 256.


#### Architecture:
+ 8 layers: 5 convolutional and 3 fully-connected, 1000-way softmax at the output.

![Architecture](images/Krizhevsky2012_architecture.png?raw=true "Architecture")

**Methodology**

+ ReLU activation function: train several times faster than tanh units.
    + Faster learning had influence on the performance of large models trained on large datasets
+ Training on Multiple GPUs
+ Local Response Normalization
    + mimics a form of lateral inhibition found on real neurons.
    + applied after ReLU in the 1st and 2nd convolutional layers.
    + improves top-1 and top-5 error rates by 1.4% and 1.2%
+ Overlapping pooling
    + Neighborhood z = 3 and stride s = 2.
    + Max-pooling employed in the 1st and 2nd convolutional layers (after response normalization) and as well as after the 5th convolutinal layer.
+ Reducing Overfitting
    + Data Augmentation
        + Generate image translations and horizontal reflections.
        + Alter the intensities of RGB channels.
    + Dropout
        + Used in the first two fully-connected layers - p(keep) = 0.5
+ Learning
    + Stochastic Gradient Descent, batch size = 128, momentum = 0.9, weight decay = 0.0005
    + Weights initialized from Gaussian distribution with mean = 0 and standard deviation = 0.01
        + Bias in 2nd, 4th, and 5th convolutional layers initialized as 1. This accelerated learning as the ReLU was fed with positive inputs from the start.
        + Bias in remaining layers initialized as zeros.
    + Learning rate ($\epsilon$)
        + Equal for all layers
        + Adjusted manually (divided by 10 when validation error stopped decreasing).
        + Initialized at 0.01 and reduced 3 times during training.

            ![Update equations](images/Krizhevsky2012_update.png?raw=true "Update equations")

    + Trained during 90 epochs (5-6 days on two NVIDIA GTX 580 3GB GPUs).

#### Results

+ Results on ILSVRC-2010 images
    + Baselines: sparse coding and Fisher vectors

Model | Top-1 | Top-5
------|-------|-------
Sparse Coding | 47.1% | 28.2%
SIFT + FVs | 45.7% | 25.7%
CNN | 37.5% | 17.0%

+ Results on ILSVRC-2012

Model | Top-1 (val) | Top-5 (val) | Top-5 (test)
------|-------|-------|-------
Sparse Coding | -- | -- | 26.2%
1 CNN | 40.7% | 18.2% | --
5 CNNs | 38.1% | 16.4% | 16.4%
1 CNN* | 39.0% | 16.6% | --
7 CNNs* | 36.7% | 15.4% | 15.3%

CNN* are convolutional neural networks pretrained on ImageNet 2011 Fall release and fine-tuned on ILSVRC-2012 training data.

+ Qualitative assessment
    + Convolutional kernels showed *specialization*

    ![Kernels](images/Krizhevsky2012_weights.png?raw=true "Convolutional kernels from 1st layer")

    + Most of top-5 labels were reasonable
    + Image similarity based on the feature activations induced at the last fully connected layer:

    ![Qualitative Assessment](images/Krizhevsky2012_qualitative.png?raw=true "Qualitative assessment")



#### Caveat:
+ Most of the choices made in the paper were based on experimental results. There is not too much theory behind.
