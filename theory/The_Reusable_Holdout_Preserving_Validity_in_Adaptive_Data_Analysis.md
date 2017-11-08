
### Paper

+ **Title**: The Reusable Holdout: Preserving Validity in Adaptive Data Analysis
+ **Authors**: Cynthia Dwork, Vitaly Feldman, Moritz Hardt, Toniann Pitassi, Omer Reingold, Aaron Roth
+ **URL**: https://pdfs.semanticscholar.org/25fe/96591144f4af3d8f8f79c95b37f415e5bb75.pdf
+ **Year**: 2015
+ **Other information**: Published in *Science* journal.
+ **Key**: Dwork2015

#### Motivation:
+ Multiple reuse of the holdout set during feature selection / feature engineering process may cause overfitting of the model in the holdout set.
+ Freedman's paradox: Picking the model based on peeking at the data ahead of time can lead to fake significant results.

#### Proposed Solution:
+ Based on differential privacy concepts.
+ The idea of *differential privacy* is to make queries to a dataset and obtain information about the data without revealing any particular records information.
  + Quoting the author: "If we can learn about the data set in aggregate while provably learning very little about any individual data element, then we can control the information leaked and thus prevent overfitting." 

#### Principles:
+ The validation can not reveal information regarding the holdout set if there is no overfit in the training set.
+ The addition of noise to the validation result is intended to prevent overfit in the holdout set.
+ A threshold is set for the maximum difference in training / validation errors.
+ The Laplacian noise parameters indicate "tolerance" and it is also directly related to the number of queries we can make in the holdout set

![Algorithm](images/Dwork2015_reusable.png?raw=true "Reusable holdout algorithm")

#### Paper example:
+ Dataset of 20,000 points, 10,000 features: 
  + Data points drawn independently from the normal distribution
  + Class is chosen uniformly at random. 
+ No correlation between the data point and its class label 
+ Reusing a standard holdout leads to accuracy of 63±0.4% (when selecting 500 out of 10,000 variables) on both the training set and the holdout set.
+ Thresholdout prevents the algorithm from overfitting to the holdout set and gives a validation error close to 50% estimate of classifier accuracy. 
+ Parameters used: T = 0.04 and tolerance = 0.01 (related to Laplacian variables).

![Results](images/Dwork2015_results.png?raw=true "Results for random features and labels")

#### Theoretical Guarantees:

+ Andy Jones's blog 'dissects' the most theoretical paper published in Arxiv and comments on the number of queries we can make in the holdout set: http://andyljones.tumblr.com/post/127547085623/holdout-reuse

+ He also provided code from the paper experiments in GitHub: https://github.com/andyljones/thresholdout/blob/master/thresholdout.py

#### Conclusions:
+ The proposed method avoids false discovery without the need to use a specialized
procedure
+ Reusable holdout gives the analyst a general and principled method to perform
multiple validation steps

#### See also:
Big Idea To Avoid Overfitting: Reusable Holdout to Preserve Validity in Adaptive Data Analysis
http://www.kdnuggets.com/2015/08/feldman-avoid-overfitting-holdout-adaptive-data-analysis.html
