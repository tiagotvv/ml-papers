### Paper

+ **Title**: Very Deep Convolutional Networks for Large-Scale Image Recognition
+ **Authors**: Karen Simonyan and Andrew Zisserman
+ **URL**: [https://arxiv.org/abs/1409.1556](https://arxiv.org/abs/1409.1556)
+ **Year**: 2015
+ **Other information**: Presented at the International Conference on Learning Representations (ICLR 2015).
+ **Key**: Simonyan2015


#### Goal:
+ Train deep convolutional neural networks with small convolutional filters to classify images into 1000 different categories.

#### Dataset
+ ImageNet Large-Scale Visual Recognition Challenge (ILSVRC): subset of ImageNet
    + 1.2 million training images, 50000 validation images, 150000 test images.
    + 1000 categories.

#### Architecture:
+ Convolutional layers followed by fully-connected layers and 1000-way softmax at the output.

![Configurations](images/Simonyan2015_architectures.png?raw=true "Convnet configurations")

+ Convolutional Layers
    + Convolutional filter: 3x3, stride = 1. 
    + 'Same' convolution, padding = 1.
    + Width of convolutional layers start at 64 and increases by a factor of 2 after max-pooling until reaching 512.
+ Max Pooling: 2x2 window, stride = 2
+ Activation function: ReLU
+ Number of parameters:
![Number of parameters](images/Simonyan2015_parameters.png?raw=true "Number of parameters")

#### Discussion:

+ A stack of three 3x3 convolutional layers (without max-pooling in between) is equivalent to a 7x7 convolutional layer. Why is it better?
    + Three non-linearities instead of just one.
    + Reduced number of parameters. A n x n convolutional layer with C channels has (nC)^2 parameters.

    
    Architecture |n | C | # of parameters
    ------|-------|----|---
    1-layer CNN | 7 | 64 | 49*4096 = 200704
    3-layer CNN | 3 | 64 | 9*4096 = 36864

+ The 1x1 convolution layers from configuration C aimed to increase the non-linearity of the decision function without affecting the the receptive fields of the convolutional layers.

#### Methodology:

+ Training
    + Optimize the multinomial logistic regression cost function.
    + Gradient descent. 
        + Mini batch size = 256, Momentum = 0.9, Weight decay = 0.0005
    + Initial learing rate: 0.01
        + Divided by 10 when the validation set accuracy stopped improving.
        + Decreased 3 times. Learning stopped after 370K iterations (74 epochs).
    + Weight initialization:
        + Configuration A was trained with random initialization of weights.
        + For the other configurations, the first convolutional nets and the fully connected nets were initialized using weights from configuration A. The other layers were randomly initialized.
        + Random initialization: weights are sampled from a zero-mean normal distribution with 0.01 variance. Biases are initialized wirh zero.
+ Reduce Overfitting:
    + Data Augmentation: followed [Krizhevsky2012](https://github.com/tiagotvv/ml-papers/blob/master/convolutional/ImageNet_Classification_with_Deep_Convolutional_Neural_Networks.md) principles with random flippings and changes in RGB levels.
    + Dropout regularization for the first two fully-connected layers - p(keep) = 0.5
+ Image Resolution:
    + Models were trained at two fixed scales S=256 and S=384.
    + Multi-scale training (randomly sampling S): minimum=256, maximum=512.
        + Can be seen as training set augmentation by scale jittering.
    + At test time, test scale Q is not necessarily equal to training scale S.

#### Results

+ Implementation derived from C++ Caffe toolbox.
+ Training and evaluation on multiple GPUs (no information regarding training time).

+ Single scale evaluation: 
    + Fixed training scale: Q=S.
    + Jittered training scale: Q=0.5(S_min + S_max).
    + Local Response Normalization did not improved results.

Configuration | S | Q | top-1 error (%) | top-5 error (%)
:--------------:|:---:|---|:-----------------:|:---------------: 
A | 256 | 256 | 29.6 | 10.4  
A-LRN | 256 | 256 | 29.7 | 10.5
B | 256 | 256 | 28.7 | 9.9
C | 256 | 256 | 28.1 | 9.4 
|  | 384 | 384 | 28.1 | 9.3
|  | [256;512] | 384 | 27.3 | 8.8 
D | 256 | 256 | 27.0 | 8.8
|  | 384 | 384 | 26.8 | 8.7
|  | [256;512] | 384 | 25.6 | 8.1 
E | 256 | 256 | 27.3 | 9.0 
|  | 384 | 384 | 26.9 | 8.7
|  | [256;512] | 384 | **25.5** | **8.0** 

+ Multi-scale evaluation: 
    + Fixed training scale: Q={S-32,S,S+32}. 
    + Jittered training scale: Q={S_min, 0.5(S_min + S_max), S_max}. 

Configuration | S | Q | top-1 error (%) | top-5 error (%)
:--------------:|:---:|---|:-----------------:|:---------------: 
B | 256 | 224,256,288 | 28.2 | 9.6
C | 256 | 224,256,288 | 27.7 | 9.2 
|  | 384 | 352,384,416 | 27.8 | 9.2
|  | [256;512] | 256,384,512 | 26.3 | 8.2 
D | 256 | 224,256,288 | 26.6 | 8.6
|  | 384 | 352,384,416 | 26.5 | 8.6
|  | [256;512] | 256,384,512 | **24.8** | **7.5** 
E | 256 | 224,256,288 | 26.9 | 8.7 
|  | 384 | 352,384,416 | 26.7 | 8.6
|  | [256;512] | 256,384,512 | **24.8** | **7.5** 

+ Dense versus multi-crop evaluation
    + Dense evaluation: fully connected layers are converted to convolutional layers at test time. Scores are obtained for full uncropped image and its flipped version and then averaged.
    + Multi-crop evaluation: average of scores obtained by passing multiple crops of the test image through the convolutional network. 
    + Combination of multi-crop and dense has best results: probably due to different treatment of convolution boundary conditions.

Configuration | Method | top-1 error (%) | top-5 error (%)
:--------------:|:---:|:-----------------:|:---------------: 
D | dense | 24.8 | 7.5
|  | multi-crop | 24.6 | 7.5
|  | multi-crop & dense | **24.4** | **7.2** 
E | dense | 24.8 | 7.5
|  | multi-crop | 24.6 | 7.4
|  | multi-crop & dense | **24.4** | **7.1** 
 

+ Comparison with State of the art solutions:

    + VGG (2 nets) = ensemble of 2 models trained using configurations D and E.
    + VGG (7 nets) = ensemble of 7 models different models trained using configurations C, D, E.

![Results](images/Simonyan2015_results.png?raw=true "Results")
