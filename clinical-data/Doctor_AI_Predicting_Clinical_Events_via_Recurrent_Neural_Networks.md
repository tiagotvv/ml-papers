### Paper

+ **Title**: Doctor AI: Predicting Clinical Events via Recurrent Neural Networks
+ **Authors**: Edward Choi, Mohammad Taha Bahadori, Andy Schuetz, Walter F. Stewart, Jimeng Sun
+ **URL**: https://arxiv.org/abs/1511.05942
+ **Year**: 2016
+ **Other information**: Presented at the Machine Learning for Healthcare Conferecence (MLHC 2016). There is also an interview about the paper at the [Data Skeptic](https://dataskeptic.com/blog/episodes/2017/doctor-ai) podcast.
+ **Key**: Choi2016



#### Goal:
+ Diagnostic and drug code prediction on a subsequent visit using diagnostic codes, medications, procedures and date of previous visits.

#### Dataset:

+ Sutter Health Palo Alto Medical Foundation - primary care - case-control study for heart failure.

![Dataset](images/Choi2016_dataset.png?raw=true "Dataset information")

+ Patients with fewer than two visits were excluded.
+ Inputs:
  + ICD-9 codes,
  + GPI drug codes
  + codes for CPT procedures
+ Records are time-stamped with the patient's visiting time.
+ If a patient receives multiple codes on the same visit, they all receive the same timestamp.
+ Granularity of codes - group subcategories:
  + ICD-9 3 digits: 1183 unique codes
  + GPI Drug class: 595 single groups
+ Target: y = [diagnosis, drug] - vector of 1183 + 595 = 1778 dimensions.

#### Architecture:
+ Gated Recurrent Units (GRU)

![GRU architecture](images/Choi2016_gru.png?raw=true "Gated Recurrent Unit (GRU)")


+ The input vector x is one-hot encoded and has a dimension of 40000. The first layer tries to reduce dimensionality.
+ Two approaches to dimensionality reduction (embedding matrix W_emb)
  + W_emb is learned together with the model.
  + W_emb is pre-trained using techniques such as word2vec.
+ Loss function: cross entropy for codes + quadratic error for forecasting visits.
+ Prediction layer codes: Softmax / Prediction layer of the next time visit: ReLu.

#### Experiments and Results:

+ Code available on GitHub: https://github.com/mp2893/doctorai
+ Implementation in Theano - Training with 2 Nvidia Tesla K80 GPUs

*Methodology*:
+ Dataset split: 85% training, 15% test.
+ RNN trained for 20 epochs.
+ L2 regularization for both the vector of coefficients of the codes and for the vector of coefficients of the next visit (lambda = 0.001) - Dropout between GRU and prediction layer (and between GRU layers if there are more than 1).
+ 2000 neurons in the hidden layer

*Baselines*:

+ Frequency: The codes from the previous visit are repeated on the new visit. Good baseline for the case of patients whose condition tends to stabilize over time.
+ Top k most frequent codes from the previous visit.
+ Logistic Regression and Multilayer Perceptron. Uses the last 5 visits to predict the next.



*Metrics*:

+ top-k recall emulates the behavior of physicians when making a differential diagnosis

    top-k recall = # of true positives in the top k predictions / number of true positives

+ R^2 used to evaluate the performance of the next visit prediction.
  + Predict logarithm of time duration between visits to reduce the impact of very long intervals.

*Results Table*:

  + RNN-1: RNN with a single hidden layer initialized with a random orthogonal matrix for W_emb.
  + RNN-2: RNN with two hidden layers initialized with a random orthogonal matrix for W_emb.
  + RNN-1-IR: RNN using a single hidden layer initialized embedding matrix w emb with the Skip-gram vectors trained on the entire dataset. 
  + RNN-2-IR: RNN with two hidden layers initialized embedding matrix W_emb with the Skip-gram vectors trained on the entire dataset.    

![Results](images/Choi2016_results1.png?raw=true "Forecasting future medical activities")


+ Performance varies according to the number of patient visits:
  + Networks learn best when they observe more records.
  + Patients with frequent visits are sicker patients. In a way, it is easier to predict the future in these cases.

![Number of visits](images/Choi2016_results2.png?raw=true "Doctor AI performance as it knows more about the patient")


+ Performance of Doctor AI in other datasets:
  + Potential to transfer knowledge accross hospitals. Pre-train Doctor AI on Sutter Health dataset and fine-tuned in MIMIC II dataset.
  
![Transfer knowledge](images/Choi2016_results3.png?raw=true "Performance of Doctor AI in other datasets")
