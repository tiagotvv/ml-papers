### Paper

+ **Title**: Network in Network
+ **Authors**: Min Lin, Qiang Chen and Suicheng Yan
+ **URL**: [https://arxiv.org/abs/1312.4400](https://arxiv.org/abs/1312.4400)
+ **Year**: 2013
+ **Other information**: Implementation using [TFLearn](https://tflearn.org) is available on [GitHub](https://github.com/tflearn/tflearn/blob/master/examples/images/network_in_network.py).
+ **Key**: Lin2013


#### Goal:
+ Propose a deep neural network structure replacing the generalized linear model structure of the convolutional filter with a multilayer perceptron (MLP).

#### Motivation:

+ Convolutional filter in CNN is a generalized linear model for the underlying data patch.
    + Implicitly makes assumption that latent concepts are linearly separable.
    + Representations that achieve good abstraction are highly non-linear functions of the training data.

#### Architecture

+ Start with a conventional CNN and replace the linear convolution filter with a "micro neural network" hence the name Network in Network (NIN).
+ Local patch is mapped to the output feature vector with a MLP consisting of fully connected layers with non-linear activation functions (ReLU in this work).
+ MLP slides over the input the same way as the convolution filters.
+ Why MLP?
    + MLP can be trained using backprop.
    + MLP can be a deep model itself.
+ This structure can also be seen as replacing a n x n convolution with a stack of 1x1 convolutions. 


![Architecture](images/Lin2013_architecture.png?raw=true "Comparison bwtween linear and mlpconv layers")

+ Global average pooling
    + Fully connected layers are prone to overfitting: replace them with a global average pooling layer.
    + Take the average of each feature map and feed it directly into the softmax layer
    + More native to the convolution structure: clarer correspondence between feature maps and categories.
    + Feature maps can be interpreted as category confidence maps.
    + No parameter to optimize.
    + Can be seen as a structural regularizer.

![Overall Structure](images/Lin2013_structure.png?raw=true "Overal Structure of NiN architecture")

#### Datasets:

+ Four benchmark datasets

| |CIFAR-10|CIFAR-100|SVHN|MNIST
----|:-----:|:-----:|:----:|:-----:
Image size | 32x32 | 32x32 | 32x32 | 28x28
Training images | 50K | 50K | 73257 + 531131(extra) | 60K
Validation images| (x) |  | (y) | |
Testing images | 10K | 10K | 26032 | 10K
Number of classes | 10 | 100 | 10 | 10

(x) in the experiments with CIFAR-10 dataset the training images are split into 40K/10K training/validation sets.

(y) 400 samples per class from training set and 200 samples per class from the extra set are used for validation.

#### Experiments and Results

+ Three stacked mlpconv layers followed by max-pooling layers.
+ Dropout is applied on the outputs of all but the last mlpconv layers.
+ Training procedure follows [Krizhevsky2012](https://github.com/tiagotvv/ml-papers/blob/master/convolutional/ImageNet_Classification_with_Deep_Convolutional_Neural_Networks.md): weight initialization and learning rates (divided by 10 as accuracy stops improving).
+ Results for this implementation are shown as *NIN+dropout* in tables.
+ CIFAR-10/CIFAR-100:
    + Pre-processing: global contrast normalization and ZCA whitening.
    + For the CIFAR-10, two hyperparameters were tuned: local receptive field size and the weight decay. For the CIFAR-100, the same settings of CIFAR-10 are used.
    + Translation and horizontal flipping augmentation.

    *CIFAR-10*

Method | Test Error (%)
------|:------------------:
Stochastic pooling | 15.13
CNN + Spearmint | 14.98
Conv. maxout + dropout | 11.68
NIN + dropout | **10.41**
CNN + Spearmint + data augmentation| 9.50
Conv. maxout + dropout + data augmentation | 9.38
DropConnect + 12 networks + data augmentation | 9.32
NIN + dropout + data augmentation | **8.81**

+ Largest activations are observed in the feature map corresponding to the groud truth category of the input image (enforced by global average pooling).

    ![Activations](images/Lin2013_activations.png?raw=true "Feature maps from last mlpconv layers")


    *CIFAR-100*

Method | Test Error (%)
-------|:--------------:
Learned pooling | 43.71
Stochastic pooling | 42.51
Conv. maxout + dropout | 38.57
Tree based priors | 36.85
NIN + dropout | **35.68**

+ SVHN - Street View House Numbers:
    + Pre-processing: local constrast normalization
    + 3 mlpconv layers followed by global average pooling

Method | Test Error (%)
-------|:--------------:
Stochastic pooling | 2.80
Rectifier + dropout | 2.78
Rectifier + dropout + synthetic translation | 2.68
Conv. maxout + dropout | 2.47
Multi-digit number recognition | 2.16
DropConnect | **1.94**
NIN + dropout | 2.35

+ MNIST:
    + Same structure as used for CIFAR-10 but number of feature maps generated from each mlpconv layer is reduced.
    + Tested without data augmentation.

Method | Test Error (%)
-------|:--------------:
2-layer CNN + 2-layer NN | 0.53
Stochastic pooling | 0.47
Conv. maxout + dropout | **0.45**
NIN + dropout | 0.47

#### Caveats
+ NIN was not tested without dropout.
+ NIN was tested only on small images - no tests were performed on the ImageNet dataset.







