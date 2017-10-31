### Paper

+ **Title**: Recurrent Neural Networks for Multivariate Time Series with Missing Values
+ **Authors**: Zhengping Che, Sanjay Purushotham, Kyunghyun Cho, David Sontag, Yan Liu
+ **URL**: https://arxiv.org/abs/1606.01865
+ **Year**: 2016
+ **Key**: Che2016

#### Motivation:
+ When sampling a clinical time series, missing values become ubiquitous due to a variety of factors such as frequency of medical events (when a blood test is performed, for example).
+ Missing values can be very informative about the label - *informative missingness*.
+ The goal of the paper is to propose a deep learning model that **exploits the missingness patterns** to enhance its performance.

![Motivation](images/Che2016_motivation.png?raw=true "Motivation")

![Time Series Notation](images/Che2016_model.png?raw=true "Time Series Notation")


#### Proposed Architecture:
+ GRU (Gated Recurrent Units) with "trainable" decays:
 + Input decay: which causes the variable to converge to its empirical mean instead of simply filling with the last value of the variable. The decay of each input is treated independently
 + Hidden state decay: Attempts to capture richer information from missing patterns. In this case the hidden state of the network at the previous time step is decayed.

![GRU architecture](images/Che2016_gru.png?raw=true "GRU architecture")

#### Dataset:
+ MIMIC III v1.4: https://mimic.physionet.org/
 + Input events, Output events, Lab events, Prescription events
+ PhysioNet Challenge 2012: https://physionet.org/challenge/2012/

| | MIMIC III | 	PhysioNet 2012 | 
|---------|--------------|----|
Number of samples (N) |	19714 |	4000 |
Number  of variables (D)	|99      |	33|
Mean number of time steps	|35.89 |	68.91|
Maximum number of time steps|150  |	155|
Mean of variable missing rate 	|0.9621|	0.8225|


#### Experiments and Results:
**Methodology**
+ Baselines:
  + Logistic Regression, SVM, Random Forest (PhysioNet sampled every 1h. MIMIC sampled every 2h). Forward / backfilling imputation. Masking vector is concatenated input to inform the models what inputs are imputed.
  + LSTM with mean imputation.
+ Variations of the proposed GRU model:
  + GRU-mean: impute average of the training set.
  + GRU-forward: impute last value.
  + GRU-simple: masking vectors and time interval are inputs. There is no imputation.
  + GRU-D: proposed model.
+ Batch normalization and dropout (p = 0.5) applied to the regression layer.
+ Normalized inputs to have a mean of 0 and standard deviation 1.
+ Parameter optimization: early stopping on validation set.

**Results**

*Mortality Prediction (results in terms of AUC)*:
+ Proposed GRU-D outperforms other models on both datasets: 
+ Random Forest and SVM are the best non-RNN baselines.
+ GRU-simple was the best RNN variant.

![Mortality Prediction](images/Che2016_results.png?raw=true "Mortality Prediction")

*Multitask Prediction (results in terms of AUC)*:
+ PhysioNet: mortality, <3 days, surgery, cardiac condition.
+ MIMIC III: 20 diagnostic categories.
+ The proposed GRU-D outperforms other baseline models.

#### Positive Aspects:
+ Instead of performing simple mean imputation or using indicator functions, the paper exploits missing values and missing patterns in a novel way.
+ The paper performs lengthy comparisons against baselines.

#### Caveats:
+ Clinical mortality datasets usually have very high imbalance between classes. In such cases, AUC alone is not the best metric to evaluate. It would have been interesting to see the results in terms of precision/recall.
