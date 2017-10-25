### Paper

+ **Title**: Clinical Intervention Prediction and Understanding using Deep Networks
+ **Authors**: Harini Suresh, Nathan Hunt, Alistair Johnson, Leo Anthony Celi, Peter Szolovits, Marzyeh Ghassemi
+ **URL**: https://arxiv.org/abs/1705.08498
+ **Year**: 2017
+ **Other information**: Presented at the Machine Learning for Healthcare Conferecence (MLHC 2017).
+ **Key**: Suresh2017


#### Goal:

Predict interventions on ICU patients using LSTM and CNN. 

#### Dataset:

MIMIC-III v.1.4 https://mimic.physionet.org/

+ Patients over 15 years of age with intensive care stay between 12h and 240h. (Only the first stay is considered for each patient) - 34148 unique records.
+ 5 static variables.
+ 29 vital signs and test results.
+ Clinical notes of patients (presented as time series).

#### Feature Engineering:

+ Topic Modeling of clinical notes: Vector of topics using Latent Dirichlet Allocation (LDA)
+ Physiological Words: Vital / Laboratory results converted to z-scores - [integer values ​​between -4 and 4] and score is one-hot encoded (each vital / lab is replaced by 9 columns). It is good idea to avoid the imputation of missing values as the physiological word in this case is the all-zero vector.

Feature vector:
+ is the concatenation of the static variables, physiological words for each vital/lab and the  topic vector.
+ 1 feature vector / patient / hour.

![Feature Vector](images/Suresh2017_FeatureVector.png?raw=true "Feature Vector")

+ 6-hour slice used to predict a 4-hour window after a 6-hour gap. All the features values are normalized between 0 and 1. (static variables are replicated).

![Prediction Interval](images/Suresh2017_prediction.png?raw=true "Prediction Interval")

#### Target Classes:

For some of the procedures to be predicted there are 4 classes:
+ Onset: Y goes from 0 to 1 during the prediction window.
+ Wean: Y goes from 1 to 0 during the prediction window.
+ Stay On: Y stays at 1 throughout prediction window.
+ Stay Off: Y stays at 0 for the entire prediction window.

#### Setup of the Experiments:

+ Dataset Split: 70% training, 10% validation, 20% test.

Long Short Term Memory (LSTM) Networks:
+ Dropout P(keep) = 0.8, L2 regularization.
+ 2 hidden layers: 512 nodes in each.

Convolutional Neural Networks
+ 3 different temporal granularities (3, 4, 5 hours). 64 filters in each.
+ Features are treated as channels. 1D temporal convolution.
+ Dropout between fully connected layers. P (keep) = 0.5.

TensorFlow 1.0.1 - Adam optimizer. Minibatches of size 128. 
Validation set used for early stopping (metric: AUC).

#### Results:

+ Baseline for comparison: L2-regularized Logistic Regression
+ Metrics: 
  + AUC per class.
  + AUC macro = Arithmetic mean of AUC per class.

+ Proposed architectures outperforms baseline.
+ Physiological words improve performance (especially on high class imbalance scenario).

![Results](images/Suresh2017_results.png?raw=true "Results")

#### Model Interpretability:
+ LSTM: feature occlusion like analysis. The feature is replaced by uniformly distributed noise between 0 and 1 and variation in AUC is computed.
+ CNN: analysis of the maximally activating trajectories.

#### Positive Aspects:
+ Relevant work: In the healthcare domain is very important to anticipate events.
+ Built on top of rich and heterogeneous data: It leverages large amounts of ICU data. 
+ The proposed model is not a complete black-box. Interpretability is crucial if the system is to be adopted in the future. 

#### Caveats:
+ Some of the methodology is not clearly explained:
  + How the split of the dataset was performed? Was it on a patient-level?
  + When testing the logistic regression baseline it is not clear how the feature vector was built. Was it built by simply flattening the 6-hour chunk?
  + For the raw data test, it was not mentioned the way the missing values were treated. 
