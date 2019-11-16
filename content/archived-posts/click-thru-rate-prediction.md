+++
title = "Avazu Click-Through Rate Prediction"
date = "2018-11-09"
+++

Draft.

## 1. Narrative (TODO)

Code published on [GitHub](https://github.com/atkm/avazu-ctr).

The data is from [a Kaggle competition](https://www.kaggle.com/c/avazu-ctr-prediction) hosted by Avazu.

## 2. Data

The columns are:

- id
- click
- hour (YYMMDDHH)
- banner\_pos
- site/app\_id/domain/category. 
- device variables: id, ip, model, type, conn\_type
- C1, C14, C15, ..., C21: anonymous features.
    Some of the anonymous features must be properties of the ad (e.g. id, category, marketer).

In most columns, hashed values are given.
(As the raw values were not salted prior to hashing, the original strings can be recovered.
See https://www.kaggle.com/c/avazu-ctr-prediction/discussion/10853 .)
Null values are also hashed, which is a nuisance, but it is possible to deduce which hashed value corresponds to null for each feature.
For example, we can use the fact that, since an ad is shown on either an app or a site, site\_id is null if and only if app\_id is not null.
Also, null is usually the most frequent value of a column.
Hashed values of null have been publicly discussed by the participants: see https://www.kaggle.com/c/avazu-ctr-prediction/discussion/10852 and https://www.kaggle.com/c/avazu-ctr-prediction/discussion/10819 .

## 3. Logistic Regression

### 3.1 Cross-validation

Negative log loss is the model evaluation metric required in the competition.

We split the data into training, validation, and test sets.
The data spans over 11 days.
Since the data is a time series, a training set must be temporally before a prediction set---especially if a feature like click history is used.
We reserve rows from the 11th day as the test set, and use the 1st-10th day for cross-validation.

| Training | Validation |
| ----: | ----: |
| 1-9 | 10 |
| 1-8 | 9, 10 |
| ... | ... |
| 1-5 | 6-10 |

The cross-validation logis is implemented in `tools.cv_tools`.

### 3.2 Feature engineering

Dealing with high-cardinality categorical features.

Encoding by count.

Encoding by average click-through rate.
This feature is useless for linear models.

Since version 0.20, sklearn's OneHotEncoder supports string features.
The ColumnTransformer class, which is also new in version 0.20, is useful for feature engineering.
A sklearn.compose.ColumnTransformer combines multiple transformers, while letting the user specify which columns are supplied to each transformer.
The outputs of the transformers are merged, and passed on to the next pipeline stage.

A custom pipeline stage is written as a subclass of sklearn.base.BaseEstimator with sklearn.base.TransformerMixin.
I wrote ClickRateEncoder and CountEncoder classes which, given a list of columns, produces the average click-through rate and counts grouped by the given columns.

## 4. The Winning Model

### 4.1 guestwalk's solution

[The winner](https://www.kaggle.com/c/avazu-ctr-prediction/leaderboard) of the competition achieved the los-loss score of 0.3791.
I referred to the following sources to understand their solution.

- [Slides](https://www.csie.ntu.edu.tw/~r01922136/slides/kaggle-avazu.pdf)
- [Paper](https://www.andrew.cmu.edu/user/yongzhua/conferences/ffm.pdf)
- [Source](https://github.com/guestwalk/kaggle-avazu)
- [Kaggle Discussion of their solution](https://www.kaggle.com/c/avazu-ctr-prediction/discussion/12608)

Although their FFM model results in the best score, the paper suggests that a logistic regression model comes close---it can achieve a log-loss score of 0.387 after the same feature engineering.

TODO: narrate [this](https://github.com/atkm/avazu-ctr/issues/12)

### 4.1 FFM

[Reading list](https://github.com/atkm/avazu-ctr/issues/11)

I use [xLearn](https://github.com/aksnzhy/xlearn) as an implementation of FFM.

### 4.2 The FFM format
FFM assumes that all columns are categorical.
In the FFM terminology, a column is referred to as a "field", and a category in a column as a "feature".
We define an FFM row to be a line in the format "label,field_1:feature_1:1,field_2:feature_2:1 ...", where field_i and feature_i are non-negative integers.
To convert a pd.DataFrame to the FFM format, define field_i := column_index, and encode categories with the same logic as sklearn's LabelEncoder to define feature_i.

An inefficient implementation of the conversion does not work for a large DataFrame.
See `tools.ffm_tools.df_to_ffm`.
There is a trade-off between space and time.
Iterating through each row + converting and writing line-by-line uses little memory, but is very slow.
pd.DataFrame.replace(dict) is fast, but uses a large amount of memory; pd.Series.map(dict) uses less memory, but it can still run into a memory issue.
Converting each entry of a DataFrame to the desired string (i.e. "field:feature:1") and writing it to a file with tocsv() maintains a good balance between space and time, although this method can also have space issue, because each column changes its dtype to "object".

### 4.3 Evaluation
The model performs far better on the "app" subset (validaton score ~= 0.34) than on the "site" subset (~= 0.44).

## 5. Conclusion and Future Work

Features of guestwalk's "base" solution that are implemented in none of the models presented above:

- Hash features instead of encoding them.
    Count features are hashed, as well.
- Thresholding count features (1000 for device\_id and device\_ip, 30 for user/hourly )
- Click history
- Hourly impression, which is defined as "concatenating all raw features together" in their slides.
    Not quite sure what that means.

Other feature ideas:

- Bucket infrequent features (say, the count is less than 10) into one "infrequent feature".
- More sophisticated count features, such as user count grouped by app\_id.
- The number of hours since the last click by the user.


Others:

- A more efficient cross-validation for xLearn.
    Currently, the model and data are read from the disk for each evaluation of a parameter.
