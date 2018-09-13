+++
title = "Prediction of NYC Taxi Demand"
date = "2018-09-11"
+++

## 1. Narrative (TODO)

Code published on [GitHub](https://github.com/atkm/taxi-demand-prediction).

## 2. Data

### 2.1 Taxi
We use the yellow cab trip CSV data available [here](http://www.nyc.gov/html/tlc/html/about/trip_record_data.shtml).
The columns that we use are the pickup location (in latitude and longitude), and pickup datetime.
The CSVs contain other columns, such as drop-off information, trip distance, and fare; there are many other analyses that are possible with this data.

For each year, there are about 170 million records, which amount to about 25GB.

| year | nRecords |
| ---- | --------:|
| 2009 | 170896055|
| 2010 | 168994353|
| 2011 | 176897199|
| 2012 | 178544324|
| 2013 | 173179759|
| 2014 | 165114361|
| 2015 | 146112989|
| 2016 | 131165043|

The format of the CSV files has changed over time.
Column names changed between 2009 and 2010, 2013 and 2014, and 2014 and 2015.
More significantly, starting in 2016/07, location IDs are used instead of latitude-longitude pairs for location data.
The website provides shape files to match location IDs with geographical areas.
In this analysis, we use the 2014 and 2015 data.

### 2.2 Weather
Our objective is to join precipitation and temperature information to the rides data.
We use [METAR](https://en.wikipedia.org/wiki/METAR) data from [New York ASOS](http://mesonet.agron.iastate.edu/request/download.phtml?network=NY_ASOS).
I wrote a Python [script](https://github.com/atkm/taxi-demand-prediction/blob/master/data/metar_data/lga_scraper.py), which is based on an example provided by the site, to obtain weather data at LGA.

We want an hourly weather information for our analysis, but the raw data contains multiple rows for every hour.
For temperature, we take the average of reported values for each hour.
For precipitation, we use the value reported at the 51st minute of each hour ([ref](https://earthscience.stackexchange.com/questions/10598/how-are-daily-precipitation-totals-computed)).

## 3. Method

### 3.1 Problem Statement
This section discusses the formal specification of our problem.
The schemas of raw taxi rides and weather data are as follows.

Rides:

| Pick-up Datetime | Pick-up Latitude | Pick-up Longitude |
| ---- | -------- | -------- |
| %Y-%m-%d %H:%M:%s | double | double |

Weather:

| Datetime | Temperature | Precipitation |
| ---- | -------- | -------- |
| %Y-%m-%d %H:%M:%s | double | double |

We only consider rides that originated in NYC, which we define to be 40.5 < latitude < 41.0 and -74.05 < longitude < -73.7.
We partition the area inside the bounds into cells of size approximately 1 km^2; this corresponds to a grid with 30 cells in the longitudinal direction and 55 cells in the latitudinal.
The following heatmap of the number of taxi pick-ups per grid cell should give the reader an intuitive idea of what our grid is.
The plot is created from a small subset (less than 5% of the entire data set) of the 2015 rides data.
The area around 4 <= grid\_x <= 8 & 24 <= grid\_y <= 30 corresponds to Manhattan.

![A plot of taxi demand by grid cell](/images/taxi-demand-prediction/demand-by-grid-cell-2015-small.png)

We then perform an aggregation of the rides table by (hour, weekday, grid cell) followed by a count operation.
The resulting schema is as follows.

Aggregated Rides:

| Pick-up Datetime | Pick-up Cell | Hour | Weekday | Count |
| ---- | -------- | -------- |-------- |-------- |
| %Y-%m-%d %H:%M:%s | (x, y)-coord of the cell | H | Mon-Sun | Int |

The weather table is aggregated by the hour to compute the average temperature and total precipitation of each hour.

Aggregated Weather:

| Datetime | Avg Temperature | Total Precipitation |
| ---- | -------- | -------- |
| %Y-%m-%d %H | double | double |


Finally, we join the weather table with the aggregated rides table based on the datetime columns, ignoring minutes and seconds.
The datetime column is dropped, as it is not used in the prediction model.

Rides and Weather Joined:

| Pick-up Cell | Hour | Weekday | Count | Avg Temperature | Total Precipitation |
| -------- | -------- |-------- |-------- | -------- | -------- |
| (x, y)-coord of the cell | H | Mon-Sun | Int | double | double |

This is the schema that the model takes as its argument.
Our goal is to predict the count column from the others.

### 3.2 Partition Into Train/Dev/Test Sets
Since our task is prediction, the training set should not contain data that we wish to perform prediction on.
We train our model on the 2014 data and predict on the 2015 data; so the training set consists of 2014 data, and the dev and test sets consist of 2015 data.
The prediction cannot be successful if the distribution of the test set differs significantly from that of the training set.
Homogeneity becomes a greater concern when including data from years further apart from 2015.
The growth of Uber and Lyft has had impacts on the ridership of NYC Taxis (the number of rides has decreased monotonically since 2013).
Some neighborhoods in Brooklyn and Bronx are seeing an increasing number of wealthier residents, who are more likely to opt for taxis rather than public transits.
Changes in the distribution of rides should be understood and reflected in the design of models.

Furthermore, there are many levels of magnification in which to study the distribution.
Here we list a few of them:

- Compare one year with another
- Compare one month of a year with another month of the same year 
- Compare one month of a year with the same month of a different year
- Study the distribution of counts in each cell
- Study the distribution of counts in each cell of the high-demand area

Here are some methods to study the homogeneity of rides data:

- Study the empirical CDF of counts at each cell. The Kolmogorov-Smirnov test is one method to do so.
- KMeans clustering
- Compute the pairwise RMSE of counts per grid cell, which is an approach that is more computationally expensive, but feasible in our case.

A rudimentary cluster analysis is done in [Demand Homogeneity ipynb](https://github.com/atkm/taxi-demand-prediction/blob/master/Demand%20Homogeneity.ipynb).

### 3.3 Prediction Target and Evaluation Metric
For a regression task, we should use either RMSE, MSE, or mean absolute error as a model evaluation metric.
(What about R-squared? Refer to Section 3 of this [lecture note](http://www.stat.cmu.edu/~cshalizi/mreg/15/lectures/10/lecture-10.pdf).)

For this project, we use the RMSE of counts scaled to [0,1] to evaluate our models.
Scaled counts are created by dividing the counts column by max(counts) taken over all (location, hour, weekday) tuples.
Using scaled counts allows for a more flexible model evaluation process.
If unscaled counts were used, the prediction depends on the size of the data set it is trained on, so the training, dev, and test sets must have the same cardinality.

To make a prediction from a scaled count, one could multiply the scaled count by the estimated max(counts).
For example, suppose we wish to predict the counts for 2015/03/01; among many others, one way to estimate the max(counts) is by using the max(counts) of 2014/03/01.
The average max(counts) in an hour is about 1200.
The scaled count multiplied by max(counts), however, is not an unbiased estimate of the unscaled count.
For a production use, the model should be trained on unscaled counts.

Another shortcoming of using scaled counts is the difficulty in computing the standard error of predictions.
If the counts were not scaled, the RMSE provides an estimate (though biased) of the standard error.
If `unscaled predictions = max(counts) * scaled predictions`, then one can show that `unscaled RMSE = max(counts) * scaled RMSE`, but, as noted in the previous paragraph, `unscaled predictions = max(counts) * scaled predictions` does not hold.

## 4. Models and Results

### 4.0 Modeling

Whether to model the variables as continuous or categorical.
Location and datetime variables need to be treated non-linearly.

### 4.1 Summary of Results

In all models, points with lower counts are over-estimated, and those with higher counts are under-estimated as shown in the residual plots in the following sections.
The baseline model, which does not use weather data, shows the best performance.
In addition, the baseline model trains much faster than the random forest model, which can have quadratic time complexity in the number of samples.

Statistics.

Baseline model.
Trained on a subset of the 2014 data of size 6,000,000 (500,000 each month * 12 months), and tested on a subset of the 2015 of the same size.

- 01m11s to train; 05m11s to predict.
- scaled RMSE:  0.031028932423957583
- RMSE:  4.0838978039637
- Residual stats (unscaled): mean -0.092445; std 4.082855
- Residual stats (scaled): mean -0.004678; std 0.030674

Random Forest:

- Parameters used: numTrees = 100; maxDepth = 30.
- More than 10 hours to train on a set of size 5,000,000.
    Since the time complexity of training can grow quadratically, training on a set of size 6,000,000 as we do for the baseline model is unrealistic.
- Prediction on a set of size 500,000 takes 32s.
- scaled RMSE: 0.041826952672794036
- Residual stats (scaled): mean -0.004084; std 0.041628


### 4.2 Baseline Model
The model and its analyses are found in [this notebook](https://github.com/atkm/taxi-demand-prediction/blob/master/Baseline%20Prediction%20Model.ipynb).

The baseline model returns the average count at each cell given hour and weekday.
This model does not use weather information.
"Aggregated Rides" in the Problem Statement section is the schema that this model takes as its argument.

This is the residual plot of this model.

![Baseline Model Residual Plot](/images/taxi-demand-prediction/baseline-train-on-2014-small-predict-on-2015-small.png)

The bias in the plot is due to the variance in counts for each (location, hour, weekday) tuple.
Naturally, most under- and over-estimated points are from the high-demand area (shown in section 3.1 of this article).
We would like to find a factor that demarcate under- and over-estimated points from accurately predicted points.

An analysis of under-estimated points suggests that including precipitation levels may improve the model.
The points with a high precipitation level is marked red in the following residual plot (the model used to generate this plot is trained on a different data).

![Residual Plot - Heavy Rain](/images/taxi-demand-prediction/baseline-model-residual-plot-heavy-rain-marked.png)

The resulting model did not perform better than the original baseline model (see notebook linked in the head of this sub-section).

Other things tried in the notebook:

- High- vs low-demand areas
- Points with extreme temperatures (very cold or hot) vs others
- Under- and over-estimated points by location, date, hour, and weekday

None of these seems to be a promising direction.


### 4.3 Random Forest Model

We did a 5-fold cross-validation on a small training set to tune numTrees, maxDepth, and minInstancesPerNode.
Changing minInstancesPerNode did not affect model performance.

Since the random forest model requires far more computational resources than the baseline model, we implemented the model in PySpark + MLlib and ran on GCP DataProc.
The DataProc image version 1.3 comes with Python 3.4, although the cluster is configured to use Python 2.7 by default.
We configured the cluster to use Python 3 with this [initialization action](https://github.com/atkm/taxi-demand-prediction/blob/master/python3-initscript.sh).


## Conclusion and Future Work

The best model is the baseline model which recorded 0.031 as its scaled RMSE.
Training a random forest model takes hundreds of times as long as the baseline model.
The average max(counts) in an hour is about 1200, so this RMSE value corresponds to a standard error of 1200 * 0.031 = 37.2 (counts) per cell per hour.

To build on the baseline model:

- Keep searching for factors that explains the variance in counts for each (location, hour, weekday) tuple.
- Make the `predict` function faster.
    In the current implementation, it looks for a (location, hour, weekday) key in a multi-index DataFrame.
    Is it the case that this implementation is effectively equivalent to a hash table lookup?
    Otherwise, re-implement the function using a hash map.
- kNN to smooth out predictions. What is an appropriate metric for the space of (location, hour, weekday) tuple?
- Interactions of variables.

To build on the random forest model:

- Try modeling the location and/or datetime variables as categoricals.
- Interactions of variables.
