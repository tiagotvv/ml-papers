### Paper

+ **Title**: Deepr: A Convolutional Net for Medical Records
+ **Authors**: Phuoc Nguyen, Truyen Tran, Nilmini Wickramasinghe, Svetha Venkatesh
+ **URL**: https://arxiv.org/abs/1607.07519
+ **Year**: 2016
+ **Other information**: Published in IEEE Transactions on Biomedical and Health Informatics.
+ **Key**: Nguyen2016

#### Goal
+ Extract features from medical records to predict future patient risk automatically.

+ Test was performed to predict unplanned readmissions 6 months after discharge.

#### Dataset:
+ Data collected from a private hospital in Australia (July/2011 - December/2015).
+ 590546 episodes (an episode is defined by an admission event and a discharge event).
  + Two separate admissions separated by less than 12h or by 12h-24h with a transfer between hospitals are defined as a single episode.
+ 300k unique patients.

#### Deepr Overview:

![Deepr Overview](images/Nguyen2016_overview.png?raw=true "Deepr Overview")

#### Sequencing:

+ Deepr initially *sequences* the EMR into a *sentence* containing
  + Diagnosis (ICD-10 AM - Australian ICD-10 variant)
  + Treatments (ACHI)
  + Time slots (encoded in 6 levels)
  + Transfers (coded as TRANSFER)
  + Other rare words are encoded as RAREWORD (<100 occurrences)

+ Here is how a *sentence* looks like:

![Deepr Sentence](images/Nguyen2016_sentence.png?raw=true "Deepr Sentence")

#### Convolutional Neural Network:

+ Words are represented as vectors.
  + One-hot encoding
  + Word embedding (dimension reduction)
+ Activation function: ReLU
+ Max-pooling
+ Motif detection: Local filter responses.

#### Outer Classifier:

+ The only requirement is that the gradient can be propagated through the layers.
  + Logistic Regression (linear)
  + Fully connected NN (non-linear)

#### Experiments and Results:

**Methodology**

+ Deepr was evaluated on a subset of the dataset
  + Training
    + 4993 patients who were readmitted without planning within 6 months of discharge (risk group)
    + 4993 patients who were not readmitted (control group) - randomly selected.
  + Validation
    + 830 patients for each of the groups
  + Testing
    + 830 patients for each of the groups
+ Words in the *sentences* may be ordered randomly or in sequence primary/secondary diagnosis, procedures.
+ For CNN, the *sentences* have a maximum of 100 words. The goal of this is to avoid skewed distribution.

+ Parameter Tuning:
  + Embedding dimensions: m = 100
  + Kernel window size: 2d + 1, d = 1
  + Motif size: 3, 4, 5; Number of motifs per size: n = 100
  + Number of epochs = 10; Mini-batch size = 64
  + Outer classifier weights (L2 regularization, lambda = 1.0)

+ Baseline model for comparison: Bag of Words + Logistic Regression (C = 0.1)

+ Target Variable: unplanned readmissions within 6 months after discharge (binary).

**Results**
+ Metric: Accuracy.

Method | without time-gaps | with time gaps
-------|-------------------|---------------
Bag of words + Logistic Regression |0.727 | 0.741
Deepr (rand init) | 0.754| 0.753
Deepr (word2vec init) | 0.750 | 0.756

+ Time-gaps information does not affect the accuracy of Deepr as well as initialization of the embedding matrix through word2vec.
+ In the Deepr feature space the patients became more aggregated and clusters were noted. One of the clusters formed are of cesarean patients who were placed together with patients who had diagnoses of complication in pregnancy and patients who performed procedures such as birth to forceps.
+ Smoother decision region.

![Deepr Decision Regions](images/Nguyen2016_decision.png?raw=true "Deepr Decision Regions")

+ Analysis of CNN filters responses. Detection of diagnostics that with strong activation in the filters and detection of motifs.

![Diagnostics with Strong Activation](images/Nguyen2016_motifs.png?raw=true "Diagnostics with Strong Activation")



#### Discussion:

+ Improve the way long-term dependencies are captured.
+ Future plans: integrate rich but unstructured clinical narrative. 

