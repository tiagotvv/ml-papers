### Paper

+ **Title**: DeepCare: A Deep Dynamic Memory Model for Predictive Medicine
+ **Authors**: Trang Pham, Truyen Tran, Dinh Phung, Svetha Venkatesh
+ **URL**: [https://arxiv.org/abs/1602.00357](https://arxiv.org/abs/1602.00357)
+ **Year**: 2016
+ **Other information**: Accepted at JBI under the new name: "Predicting healthcare trajectories from medical records: A deep learning approach".
+ **Key**: Pham2016

#### Goal:
+ Build a deep dynamic neural network that reads medical records, stores previous illness history, infers current illness states and predicts future medical outcomes. 
 
#### Dataset:

+ Data collected over 12 years in an Australian hospital (2002-2013)
+ Selection criteria:
    + Admissions with incomplete patient information were eliminated
    + Patients with less than 2 admissions were eliminated

+ Diagnostic and procedural vocabulary:
    + Diagnostics that have the first two equal characters are collapsed into a single diagnosis
    + Procedures with the first 3 digits are collapsed in a single procedure.

+ Two cohorts: Diabetes and Mental Health

| | Diabetes | Mental Health
---|----|----|
Number of patients (% of males, median age) | 12,000 (55%, 73 years) | 11,000 (49.4%, 37 years)
Number of patients after selection criteria | 7191 | 6109
Number of admissions | 53208 | 52049
Number of diagnostic codes| 243 | 247
Number of procedure codes| 773 | 753
Number of drug codes| 353 | 319


#### Architecture:

+ DeepCare is a deep dynamic neural network that has three main layers. 
    + Bottom layer is a LSTM: cells are modified to handle irregular timing and interventions. 
        + Input is a sequence of admissions. Each admission contains a set of diagnosis codes (which is then formulated as a feature vector x_t), a set of intervention codes (which is further formulated as a feature vector p_t), the admission method mt e {1,2} and the elapsed time.
    + Modifications to LSTM structure to capture desired effects
        + The input gate takes into account whether or not the admission is planned.
        + The output gate takes into account the current state of the intervention (p_t) while the forgetting gate is influenced by the state (p_t-1)
        + Temporal irregularities are placed in the forgetting gate as a function of the time interval passed between steps t-1 and t. The paper suggests a vector of 3 components modeling at 60, 180 and 365 days.
    + The middle layer aggregates illness states through multiscale weighted pooling.
        + Max-pooling admission: resembles the usual coding practice that one diagnosis is picked as the primary reason for admission.
        + Sum-pooling admission: with multiple diseases (multiple comorbidities) is more likely to be at risk than those with single condition.
        + Mean-pooling admission: in absence of primary conditions, a mean pooling could
be a sensible choice
    + The top layer is a fully connected neural network that takes pooled states and other statistics to estimate the final outcome probability.

![Deep Care Architecture](images/Pham2016_architecture.png?raw=true "Deep Care Architecture")


+ The vectors x_t and p_t are obtained through embedding to reduce dimensionality

![Deep Care Architecture](images/Pham2016_embedding.png?raw=true "Deep Care Architecture")

+ After LSTM: multiscale pooling and emphasis on the most recent events.

+ Vector containing pooled states with lookbacks of 12 months, 24 months and of all clinical history.

#### Experiments and Results:

*Methodology*
+ Split of datasets: 2/3 training, 1/6 validation, 1/6 validation
+ Variation of the number of embedding dimensions (M) and neurons in the hidden layer (K): between 5 and 50.
+ For disease progression and recommendation for intervention, the best values ​​were M = 30 and K = 40.
+ For prediction of future risk, M = 10, K = 20.
+ Stochastic gradient with variable learning rate
    + starts at 0.01. If the model does not find a point with a lower cost function, we expect *n_wait* epochs until we divide learning rate by 2.
    + Initially *n_wait* = 5, after the first modification, *n_wait* = min (15, *n_wait* + 2)
    + The learning ends when *n_epochs* = 200 or when learning rate is <0.0001.


*Metric*

+ Precision at K (Precision@K):
    + Percentage of relevant results in retrieved results. That means if the model predicts *np* diagnoses of the next readmission and *nr* diagnoses among of them are relevant the model's performance is 
    
        Precision@*np*  = *nr/np* 

*Results*

+ Predicting the next *np* diagnoses that a patient will receive.
    + Baselines: Markovian model and conventional RNN

    + Table shows Precision@*np* (%) for *diabetes - mental health* datasets.

Model | *np*=1 | *np*=2 | *np*=3
------|:--------:|:--------:|:--------:
Markov | 55.1 - 9.5  | 34.1 - 6.4 | 24.3 - 4.4
Plain RNN | 63.9 - 50.7 | 58.0 - 45.7 | 52.0 - 39.5
Deep Care (mean adm) | **66.2** - **52.7**  | **59.6** - **46.9** | **53.7** - **40.2**
Deep Care (sum adm) | 65.5 - 51.7| 59.3 - 46.2  |53.5 - 39.8
Deep Care (max adm) | 66.1 - 51.5 | 59.2 -46.7 | 53.2 - **40.2**

+ Predict the next procedures for a patient:

    + Table shows Precision@*np* (%) for *diabetes - mental health* datasets.

Model | *np*=1 | *np*=2 | *np*=3
------|:--------:|:--------:|:--------:
Markov | 35.0 - 20.7  | 17.6 - 12.2 | 11.7 - 8.1
Plain RNN | 77.7 - 70.4 | 54.8 - 55.4 | 43.1 - 43.7
Deep Care (mean adm) | 77.8 - 70.3  | 54.9 - 55.7 | 43.3 - 44.1
Deep Care (sum adm) | **78.7** - **71.0** | **55.5** - **55.8**  | **43.5** - **44.7**
Deep Care (max adm) | 78.4 - 70.0 | 55.1 - 55.2 | 43.4 - 43.9

+ Unplanned readmission prediction and at-risk patients
    + Results of unplanned readmission prediction in F-score (%) within 12 months for diabetes and 3 months for mental health (DC is DeepCare, int is intervention)
    + Shows gain when modeling the irregular timing, interventions and recency+multiscale pooling, as well as parametric time.

Model | Diabetes | Mental Health
------|:--------:|:---------:
SVM (max-pooling) |64.0 |64.7 
SVM (sum-pooling) |66.7 |65.9 
Random Forests (max-pooling) |68.3 |63.7 
Random Forests (sum-pooling) |71.4 |67.9 
Plain RRN (logist. regress.) |75.1 |70.5 
LSTM (logit. regress.) |75.9 |71.7 
DC (nnets + mean adm.) |76.5 |72.8 
DC ( [int + time decay] +recent.multi.pool. +nnets+mean adm.) |77.1 |74.5 
DC ( [int + param. time) +recent.multi.pool.+nnets+mean adm.) | **79.0** | **74.7** 
