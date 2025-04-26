# Power Outage Analysis

**Author**: Zhongyan Luo

## Step 1: Introduction

​Power outages have become an increasingly pressing concern due to their frequency and impact on modern society. Recent data indicates that the United States has experienced a significant rise in weather-related power outages during warmer months, with a 60% increase over the past decade compared to the 2000s. This uptick is largely attributed to climate change, which intensifies storms, wildfires, and other extreme weather events, placing additional stress on aging power infrastructure. Beyond the immediate inconvenience, outages pose serious risks to public health and safety. For instance, individuals reliant on medical devices such as ventilators face life-threatening situations during prolonged outages. Economically, power outages can be devastating. Businesses suffer from operational disruptions, leading to great financial losses.

This dataset contains detailed information about power outage occured across states in US. It includes data on the time & location of outages, environmental and demographic factors, impacted population, etc. Some key features include climate-related anomalies, population density, land and water percentage, and other attributes that may influence power outages. The dataset consists of 1,534 records and 56 columns.

By researching the dataset, we can get some insight abou the following questions:
- How have power outages changed over the years? Are they becoming more frequent or severe?
- Which states or regions experience the most outages? Is there a pattern based on location?
- How do climate anomalies (e.g., cold vs. warm) correlate with the frequency and duration of outages?
- Are there specific months or seasons when outages are more likely to occur?

In the predictive analysis of this project, I will predict the causes of power outage based on other related factors.

## Step 2: Data Cleaning and Exploratory Data Analysis

### Data Cleaning
To make sure we can modeling the data conviniently and correctly, we clean the data by the following steps:

- Replaced missing numerical values (e.g., MONTH, ANOMALY.LEVEL, OUTAGE.DURATION, CUSTOMERS.AFFECTED) with NaN for proper handling.
- Converted empty categorical values (e.g., CLIMATE.REGION, CLIMATE.CATEGORY, CAUSE.CATEGORY.DETAIL, HURRICANE.NAMES) to empty string.
- Converted OUTAGE.START.DATE and OUTAGE.RESTORATION.DATE to datetime format.

### Univariate Analysis

Then, to get a brief understanding of the data, we explore if there's trend or interesting pattern in data.
First, lets take a look at the most common causes of major outages.
<iframe
  src="assets/fig_cause.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>The graph shows that the most common cause is severe weather, followed by intentional attack, system operability distruption, public appeals and so on.

We also want to see if there's trend of outage over years.
<iframe
  src="assets/fig_year.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>The number of outage over years has an increasing trend until 2011 where it reaches the peak, and after 2011, the number decreases gradually.

### Bivariate Analysis

We are curious about if different causes of outage can have different durations. Lets plot their relationship using a boxplot.
<iframe
  src="assets/fig_year.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>The boxplot highlights the distribution of outage duration across different cause categories, which can reveal patterns in how different types of outages vary in duration.

### Grouping and Aggregates

Intuitively, in the same time period, if the anormaly level (the cold and warm episodes by season) is higher, outage might occur more frequently due to unstable weather condition.
We do a double grouping here and see if that guess is true.
<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>MONTH</th>
      <th>ANOMALY.LEVEL</th>
      <th>OBS</th>
      <th>PCT_LAND</th>
      <th>PCT_WATER_TOT</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>-1.6</td>
      <td>2</td>
      <td>93.88</td>
      <td>6.12</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>-1.4</td>
      <td>6</td>
      <td>86.91</td>
      <td>13.09</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>-1.3</td>
      <td>14</td>
      <td>83.78</td>
      <td>16.22</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>-0.7</td>
      <td>33</td>
      <td>94.14</td>
      <td>5.86</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>-0.5</td>
      <td>38</td>
      <td>91.50</td>
      <td>8.50</td>
    </tr>
  </tbody>
</table>
</div>
The table shows a clear trend that with higher anormaly level, outage tends to occur more often.


## Step 3: Assessment of Missingness

### NMAR Analysis

One column in the dataset that may be Not Missing At Random (NMAR) is "HURRICANE.NAMES". This column has a significant number of missing values (1462), which suggests that missingness is likely due to the nature of the outage cause. If a power outage was not caused by a hurricane, it is reasonable that the hurricane name would be missing.

### Missingness Dependency

Since the column "CUSTOMERS.AFFECTED" has the most missing values, we will check its missingness dependency.
By running a permutation test, we get the following p-values:
<div>
<table border="1" class="dataframe">
  <tbody>
    <tr>
      <th>CLIMATE.CATEGORY</th>
      <td>0.178</td>
    </tr>
    <tr>
      <th>CAUSE.CATEGORY</th>
      <td>0.0</td>
    </tr>
    <tr>
      <th>RES.PRICE</th>
      <td>1.0</td>
    </tr>
    <tr>
      <th>TOTAL.PRICE</th>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
</div>

Based on the permutation tests:

The missingness of "CUSTOMERS.AFFECTED" depends on "CAUSE.CATEGORY" (p-value = 0.0). This suggests that the likelihood of missing values in "CUSTOMERS.AFFECTED" is influenced by the cause of the outage. Certain outage causes may not require or report the number of affected customers, leading to systematic missingness.

The missingness of "CUSTOMERS.AFFECTED" does not depend on "RES.PRICE" (p-value = 1.0) and "TOTAL.PRICE" (p-value = 1.0). This indicates that the likelihood of missing values in "CUSTOMERS.AFFECTED" is independent of electricity prices, meaning the missingness is likely unrelated to economic factors.

The missingness of "CUSTOMERS.AFFECTED" may not depend on "CLIMATE.CATEGORY" (p-value = 0.178). This suggests weak evidence of dependence, meaning climate conditions are not a strong determinant of missing customer impact data. See the following graph.
<iframe
  src="assets/fig_missing.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

## Step 4: Hypothesis Test

Null Hypothesis:
The average number of customers affected by severe weather outages is the same as the average number of customers affected by intentional attack outages.

Alternative Hypothesis:
There is a difference between the average number of customers affected by severe weather outages and the average number of customers affected by intentional attack outages.

p-value < 0.05. We reject the null hypothesis. 
It is likely that we can use the number of affected customers to predict the reason of outage.

## Step 5: Framing a Prediction Problem

Our model is a classifier of multiclass classification. The objective of this project was to develop a machine learning model capable of predicting the cause of major power outages based on historical data. In addition to building the predictive model, a fairness analysis was conducted to determine whether the model performed differently across urban and rural regions. This analysis was important to ensure that the model does not disproportionately underperform for any particular group. 

A Random Forest Classifier was trained to predict the cause of power outages, and its performance was evaluated using standard classification metrics. The model achieved an accuracy of 65.8% and f-1 score [0.11, 0.75, 0.1 , 0.47, 0.78, 0.17], with relatively good performance in predicting causes such as vandalism and hurricanes, but weaker performance for less common causes like tornadoes and wind storms. The imbalance in class distribution presented a challenge, as certain categories had very few samples, leading to poor recall in those groups. Then, we add several more features and tune the hyperparameter to improve our baseline model and achieve an accuracy of 78.2%.

While accuracy gives a high-level view of model performance, F1-score would be more useful if we are particularly concerned with the model's ability to correctly classify underrepresented outage causes. If fairness is a concern (e.g., ensuring the model doesn’t disproportionately misclassify outages in rural areas), using F1-score for individual classes could provide deeper insights. The F-1 score of our final model is [0.13, 0.86, 0.53, 0.64, 0.89, 0.29], which indicates its performance is better for gruops that has more samples.

## Step 6: Baseline Model

Since we use a random forest model, we do not prefer to use any encoding techniques due to the structure of trees (encoding will increase number of features and thus increase depth of tree).
To predict the cause of outage, we first choose columns 'YEAR' (quantitative), 'MONTH' (quantitative), 'CLIMATE.REGION' (nominal), 'ANOMALY.LEVEL' (quantitative), 'CLIMATE.CATEGORY' (nominal), 'RES.CUSTOMERS' (quantitative).
The model achieved an accuracy of 65.8% with f-1 score [0.11, 0.75, 0.1 , 0.47, 0.78, 0.17]. The performance is not that good since it's just a bit higher than pure guessing (50%). We might need more features to get a better prediction.

## Step 7: Final Model

To improve the model, we add features 'CUSTOMERS.AFFECTED' (quantitative), 'PC.REALGSP.STATE' (quantitative), 'POPPCT_URBAN' (quantitative).
Since different kind of cause will lead to different number of people affected (discussed in step 4), more affected people might indicate a severe weather cause and less affected people might indicate an intentional attack. We can count the districts or family affected to get the'CUSTOMERS.AFFECTED' value after outage and use it to do prediction.
Also, GSP can be another indicator since states with higher GSP tend to have better security and equipment to avoid intentional attack and equipment failure.
I choose population percentage in urban area as well. If one state has more urban residents, the pressure of electricity can be higher and it is more difficult to manage the power system, leading to system operability disruption.

I used GridSearchCV for tuning the hyperparameters. The best hyperprameters are:

- 'classifier__max_depth': 20
- 'classifier__min_samples_leaf': 1
- 'classifier__min_samples_split': 5
- 'classifier__n_estimators': 300

The accuracy increases from 65.8% to 78.2% and F-1 score becomes [0.13, 0.86, 0.53, 0.64, 0.89, 0.29], which indicates a significant improvement in the prediction performance.

## Step 8: Fairness Analysis

To assess fairness, the model's precision was compared for urban and rural areas, using the percentage of urban population as a proxy. A permutation test was conducted with 1,000 iterations to evaluate whether the observed difference in precision scores between urban and rural groups was statistically significant. 

Null Hypothesis:
The model's precision is the same for both urban and rural areas. Any observed difference in precision is due to random chance.

Alternative Hypothesis:
The model's precision is significantly different between urban and rural areas.

The test statistics is the difference between prediction precision of urban and rural areas.
By running the test, we get p-value 0.091 > 0.05. We fail to reject the null hypothesis.
