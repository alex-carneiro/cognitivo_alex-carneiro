# README

Analysis of _winequality.csv_ file.

*Objective*: to build a model that predicts wine quality, as score in 0 ~ 10 (integer value), based on the following physico-chemical properties:

* pH
* free sulfur dioxide
* sulphates
* density
* fixed acidity
* residual sugar
* volatile acidity
* citric acid
* type
* total sulfur dioxide
* chlorides
* alcohol

## Answers

### a) How was defined your modeling strategy?

I have based my modeling strategy on CRISP-DM and KNN methodologies. First of all, I looked for dirty data and missing values on the dataset. There are no missing values and I removed dirty samples.

I saw there is a pattern for dirty samples in *alcohol* field, in which they have dots for each three digits and most likely position of the dot is between second and third digits. But there was only 40 dirty samples and this kind of assumption is not always safe, so that I decided to remove dirty samples.

Second step was to understand how each field looks like and their distributions.

Third step was obtaining the correlation matrix for the fields and the correlation between each field and the expected output.

Whether correlation matrix presented many high values, a PCA transformation might improve results. But, I decided not to use it because correlation matrix presented most of values below _0.3_.


### b) How was defined employed loss function?

I chose three algorithms for tests, two ones are based on least squares technique (Linear Regression and SVM) and the other one is a clustering algorithm adapted to perform regression (KNN).

Each algorithm was tested with raw data and normalized data. Evaluation metrics include Mean Absolute Error (MAE), Accuracy and Error lower than 1.


### c) What was your criteria for choosing final model?

I chose SVR algorithm with normalized data because it presented lowest MAE, highest Accuracy and highest robustness to minor errors.


### d) What was your criteria for model evaluation? Why did you choose using this method?

I looked first for models with low MAE and high Accuracy. But, after thinking about the problem, I decided to consider errors of less than 1 and robustness to minor errors as well.

I chose those metrics for model evaluation because this problem contains a subjective component that means the model should be able to predict the feeling of a consumer based on physico-chemical properties. Thus, an error of 1 unit sounds tolerable for me.

I decided not to perform a classification approach because the output values are sequentially related, and classification approaches use to give better results for problems in which expected output values do not present relations, like a sequence.


### e) Which evidence do you have for ensuring your model is good enough?

My final model is able to predict the wine quality with an expected error of 0.452, it means less than 1 error of 1 unit by 2 samples of wine in average.

Another evidence is that expected error be less or equal to 1 is 94.24%, it looks a great confidance for a prediction within the error that I supposed to be tolerable.

Last evidence that I collected is that given the error is lower or equal to 1, there is 64.4% of probability of the prediction is correct.


The following of this document explains how I achieved this results.


## Data exploration

File _First Contact.ipynb_ contains the first exploratory analysis performed on the dataset.

### Insights

The dataset has 6497 samples of wine.

*quality* values are in the range _3 ~ 9_.

*alcohol* field is read as string because some values have more than one dot, e.g.: '128.933.333.333.333'. Those values have been removed.

All fields present values in different ranges. This means that a normalization might improve predictions. The chosen normalization performed in the dataset makes _mean(field)=0_ and _variance(field)=1_, but it does not change histogram shapes neither correlation between fields.

*type* field is binary {Red, White}, so that it has been mapped to {0,1}, respectively.

Most of the fields present low correlation, only *total sulfur dioxide* and *free sulfur dioxide* has presented correlation above _0.7_.

All of the fields are low correlated to the output, it is observed highest correlation with *alcohol* that presents _0.44_.


## Modeling

File _Regression Approach.ipynb_ contains the experiments performed on the dataset with different regression algorithms.

As the expected output is a score, a regression approach is convenient for minimizing the difference between the real value and the predicted one.

The three regression algorithms were:

* Linear Regression;
* SVR (regression based on SVM);
* KNN Regression.

Those three algorithms were evaluated with raw and normalized data.

Rate for training/testing was 90%/10%.

Metrics for evaluating the methods were:

* _MAE_: Mean Absolute Error.
* _Accuracy_: Rate of correctly predicted outputs.
* _Error=1_: Rate of predicted outputs in which observed error is 1, e.g.: real quality is 6 but model predicts 7 or 5.
* _Error<=1_: Rate of predicted outputs in which observed error is lower or equal to 1, it means _Accuracy + Error=1_.

### Linear Regression results

|Metrics |Raw data|Normalized data|
|--------|--------|---------------|
|MAE     |0.558   |0.558          |
|Accuracy|48.92%  |48.92%         |
|Error=1 |43.96%  |43.96%         |
|Error<=1|92.98%  |92.98%         |


### SVR results

|Metrics |Raw data|Normalized data|
|--------|--------|---------------|
|MAE     |0.497   |0.469          |
|Accuracy|56.04%  |59.75%         |
|Error=1 |38.70%  |33.75%         |
|Error<=1|94.74%  |93.50%         |

### KNN Regression results

|Metrics |Raw data|Normalized data|
|--------|--------|---------------|
|MAE     |0.597   |0.534          |
|Accuracy|47.83%  |52.94%         |
|Error=1 |45.20%  |40.87%         |
|Error<=1|93.03%  |93.81%         |


## Model evaluation

The SVR model with normalized data has been chosen because it presented lowest observed _MAE_ (0.469), highest observed _Accuracy_ (59.75%) and highest rate _Accuracy/(Accuracy + (Error=1))_ (0.639). The last metric means this model is more robust to minor errors, this value makes this model better than SVR with raw data although that SVR with raw data presented a higher value for _Error<=1_ than SVR with normalized data.

SVR model has been evaluated with a KFold method, in which the dataset was divided into 10 subsets and each one was used as testing while the other 9 were used as training. This method produced 10 predictions, one for each subset, and the final evaluation was obtained by the mean of those predictions:

* Max MAE = 0.474
* Min MAE = 0.423
* Mean MAE = 0.452
* Var of MAE = 0.00028
* Mean Accuracy = 60.88%
* Mean Error=1 = 33.54%

### Conclusion

SVR model trained with normalized data is able to predict wine quality with expected absolute error of 0.452, accuracy of 60.88%, expected error lower or equal to 1 is 94.24% and rate _Accuracy/(Accuracy + (Error=1))_ = 0.644.
