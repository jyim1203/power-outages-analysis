Project hosted on GitHub [here](https://github.com/jyim1203/power-outages-analysis/tree/main)!


# Power Outages Analysis
UCSD DSC80 Project
by Jonathan Yim (jgyim@ucsd.edu)

---

## Introduction

In this project, I analyzed a dataset of major U.S. power outages from January 2000 to July 2016, sourced from Purdue University’s Laboratory for Advancing Sustainable Critical Infrastructure at https://engineering.purdue.edu/LASCI/research-data/outages. Each outage meets the Department of Energy’s criteria of impacting at least 50,000 customers or causing an unplanned energy demand loss of at least 300 MegaWatts.

My analysis will proceed in four main stages:

1. Data Cleaning and Exploratory Analysis to understand the dataset's structure and distributions.

2. Missingness Analysis to examine the mechanisms and dependencies of missing data.

3. Hypothesis and Permutation Testing to rigorously assess specific relationships within the data.

4. Predictive Modeling to forecast the duration (length) of an outage.

First, I will clean the data and visualize key features to uncover initial patterns. My core research question is: How long can we expect a major power outage to last based on information typically available to an impacted individual or utility? I will build a model to predict outage duration using accessible features such as the time of occurrence (month, year, hour), location (state, climate region), and more, while withholding the exact cause of the outage. Accurately predicting outage length is crucial for utilities to optimize response efforts and resource allocation, and for communities and emergency services to improve preparedness and mitigate economic and public safety impacts. From the impacted consumer's point of view, it may also help individuals make informed decisions, from how to prepare for longer outages, staying safe without power, reducing anxiety and worries, and more.

The dataset contains 1,534 outages (rows) described by 57 features (columns), including information on outage characteristics, geography, climate, and state-level economic statistics. For this analysis, I will focus on a curated subset of these features.

| Feature Name | Type | Group | Description |
|:---|:---|:---|:---|
| **--- Outage Timing & Target ---** | | | |
| `outage_duration` | Numeric | Target | The length of the outage in minutes |
| `year` | Categorical | Timing | The year the outage started. |
| `month` | Categorical | Timing | The month the outage started. |
| `outage_start_date` | Date/Time | Timing | Date the outage began. |
| `outage_start_time` | Date/Time | Timing | Time the outage began. |
| `outage_restoration_date` | Date/Time | Timing | Date the power was restored. |
| `outage_restoration_time` | Date/Time | Timing | Time the power was restored. |
| **--- Outage Impact & Cause ---** | | | |
| `customers_affected` | Numeric | Impact | The number of customers who lost power. |
| `demand_loss_mw` | Numeric | Impact | The peak power loss in MegaWatts (MW). Some cases have total demand reported |
| `cause_category` | Categorical | Cause | High-level reason for the outage (e.g., Weather, Equipment Failure). |
| **--- Geographic & Economic Context ---** | | | |
| `us_state` | Categorical | Geography | The US state where the outage occurred. |
| `nerc_region` | Categorical | Geography | North American Electric Reliability Corporation (NERC) regions involved in the outage event |
| `climate_region` | Categorical | Climate | U.S. Climate regions as specified by National Centers for Environmental Information (e.g., Southeast, West). |
| `climate_category` | Categorical | Climate | General climate category (e.g., Cold, Tropical). |
| `anomaly_level` | Categorical | Climate | Categorical level of weather anomaly at the time of outage referring to the Oceanic El Nino/La Nina (ONI) Index. |
| `total_customers` | Numeric | Demographics | Total utility customers in the state. |
| `population` | Numeric | Demographics | Total population in the state. |
| `poppct_urban` | Numeric | Demographics | Percentage of state population in urban areas. |
| `popden_urban` | Numeric | Demographics | Population density in urban areas (persons per square mile). |
| `areapct_urban` | Numeric | Demographics | Percentage of state area classified as urban. |
| `total_price` | Numeric | Economics | Average monthly electricity price in the U.S. state (cents/kilowatt-hour) |
| `total_sales` | Numeric | Economics | Total electricity sales in the state (megawatt-hour). |
| `util_contri` | Numeric | Economics | Utility's total value contribution to the state's economy. |
| `util_realgsp` | Numeric | Economics | Utility's contribution to Real Gross State Product (GSP). |
| `total_realgsp` | Numeric | Economics | Total Real GSP for the state. |

---

## Data Cleaning and Exploratory Data Analysis
Our first step is to clean the data and make it more suitable for analysis.

### Data Cleaning
1. I start by dropping irrelevant columns, keeping only the features of interest listed below for analysis:     
    'year',
    'month',
    'us_state',
    'nerc_region',
    'climate_region',
    'climate_category',
    'anomaly_level',
    'outage_start_date',
    'outage_start_time',
    'outage_restoration_date',
    'outage_restoration_time',
    'outage_duration',
    'cause_category',
    'demand_loss_mw',
    'customers_affected',
    'total_price',
    'total_sales',
    'total_customers',
    'poppct_urban',
    'popden_urban',
    'areapct_urban',
    'util_realgsp',
    'total_realgsp',
    'util_contri',
    'population'
2. Then, I make sure the columns are of the right data types and drop duplicate observations (10 observations dropped).
3. Next, I combined the 'outage_start_date' and 'outage_start_time' columns into one Timestamp object in an 'outage_start_datetime' column. I do the same for 'outage_restoration_date' and 'outage_restoration_time' with the new colun 'outage_restoration_datetime'. I then dropped the old columns since all the relevant information is in 'outage_start_datetime' and 'outage_restoration_datetime'.
4. After creating valid DateTime columns, I recalculated the outage_duration column using 'outage_start_datetime' and 'outage_restoration_datetime' to get rid of estimates or inexact duration times, also checking for negative (impossible) durations.
5. Finally, we check to make sure the data is tidy and the categories have consistent variable naming (no duplicates due to spelling, capitalization, punctuation, etc).
A sample of our dataset is available below:

<div style="overflow-x: auto;">

|   year |   month | us_state   | nerc_region   | climate_region     | climate_category   |   anomaly_level |   outage_duration | cause_category     |   demand_loss_mw |   customers_affected |   total_price |   total_sales |   total_customers |   poppct_urban |   popden_urban |   areapct_urban |   util_realgsp |   total_realgsp |   util_contri |   population | outage_start_datetime   | outage_restoration_datetime   |
|-------:|--------:|:-----------|:--------------|:-------------------|:-------------------|----------------:|------------------:|:-------------------|-----------------:|---------------------:|--------------:|--------------:|------------------:|---------------:|---------------:|----------------:|---------------:|----------------:|--------------:|-------------:|:------------------------|:------------------------------|
|   2011 |       7 | Minnesota  | MRO           | East North Central | normal             |            -0.3 |              3060 | severe weather     |              nan |                70000 |          9.28 |   6.56252e+06 |       2.5957e+06  |          73.27 |           2279 |            2.14 |           4802 |          274182 |       1.75139 |  5.34812e+06 | 2011-07-01 17:00:00     | 2011-07-03 20:00:00           |
|   2014 |       5 | Minnesota  | MRO           | East North Central | normal             |            -0.1 |                 1 | intentional attack |              nan |                  nan |          9.28 |   5.28423e+06 |       2.64074e+06 |          73.27 |           2279 |            2.14 |           5226 |          291955 |       1.79    |  5.45712e+06 | 2014-05-11 18:38:00     | 2014-05-11 18:39:00           |
|   2010 |      10 | Minnesota  | MRO           | East North Central | cold               |            -1.5 |              3000 | severe weather     |              nan |                70000 |          8.15 |   5.22212e+06 |       2.5869e+06  |          73.27 |           2279 |            2.14 |           4571 |          267895 |       1.70627 |  5.3109e+06  | 2010-10-26 20:00:00     | 2010-10-28 22:00:00           |
|   2012 |       6 | Minnesota  | MRO           | East North Central | normal             |            -0.1 |              2550 | severe weather     |              nan |                68200 |          9.19 |   5.78706e+06 |       2.60681e+06 |          73.27 |           2279 |            2.14 |           5364 |          277627 |       1.93209 |  5.38044e+06 | 2012-06-19 04:30:00     | 2012-06-20 23:00:00           |
|   2015 |       7 | Minnesota  | MRO           | East North Central | warm               |             1.2 |              1740 | severe weather     |              250 |               250000 |         10.43 |   5.97034e+06 |       2.67353e+06 |          73.27 |           2279 |            2.14 |           4873 |          292023 |       1.6687  |  5.48959e+06 | 2015-07-18 02:00:00     | 2015-07-19 07:00:00           |
</div>

### Exploratory Data Analysis

#### Univariate Analysis
In my EDA, I first focus on univariate analysis to better understand the distribution of certain variables of interest.

First, we examine the counts of outages over time.
<iframe src="https://jyim1203.github.io/power-outages-analysis/plots/annual_outages_line.html" width=800 height=600 frameBorder=0></iframe>

Then, we have the distribution of major outages across different cause categories.
<iframe src="https://jyim1203.github.io/power-outages-analysis/plots/outages_causes.html" width=800 height=600 frameBorder=0></iframe>

Next, I plotted the number of outages by US state.
<iframe src="https://jyim1203.github.io/power-outages-analysis/plots/outages_by_state.html" width=800 height=600 frameBorder=0></iframe>

Finally, I plotted the distribution of outage durations.
<iframe src="https://jyim1203.github.io/power-outages-analysis/plots/raw_outage_duration.html" width=800 height=600 frameBorder=0></iframe>

Since this plot had a very heavy skew, I also plotted the log-transformed outage durations.
<iframe src="https://jyim1203.github.io/power-outages-analysis/plots/log_outage_duration.html" width=800 height=600 frameBorder=0></iframe>

#### Bivariate Analysis

To further examine the relationship between outage duration and the cause of the outage, I created the violin plot below. This also helps us better picture the frequency of some of the outage events, as well as get a better picture of the median duration of outages from each cause category. It also helps show how some of the longest outage durations are within the fuel supply emergency and equipment failure categories. 
<iframe src="https://jyim1203.github.io/power-outages-analysis/plots/vio_duration_by_cause.html" width=800 height=600 frameBorder=0></iframe>

There appears to be a positive correlation, with the number of customers affected growing as the number of total customers increases as well. However, there are still many cases in which the number of customers affected exceeds the number of total customers in the state. This may either be a data quality issue, or perhaps the outage spans greater areas, spilling into other states or populous regions. However, this happens very infrequently, with the majority of observations having less customers affected than total customers (as expected). 
<iframe src="https://jyim1203.github.io/power-outages-analysis/plots/customers_total_vs_affected.html" width=800 height=600 frameBorder=0></iframe>

Examining the relationship between Outage Duration and Customers Affected, we notice a cluster where typical outages affect between roughly 40,000 and 400,000 customers, lasting between 100 and 10,000 minutes. Overall, most outages seem to be confined within this range of moderate magnitude and duration, although the vertical spread indicates duration can vary significantly across observations. Notably, there is a lack of a strong positive correlation, which we would expect with major outages affecting many customers also having a longer duration, but there may be a weaker relationship present.
<iframe src="https://jyim1203.github.io/power-outages-analysis/https://jyim1203.github.io/power-outages-analysis/plots/duration_vs_affected.html" width=800 height=600 frameBorder=0></iframe>

#### Grouping and Aggregate Analysis

Here I examined the average severity metrics per NERC region, aggregating using the mean metrics for each region. As expected, larger, populus regions tend to have higher numbers of customers affected. However, outage durations vary wildly, perhaps due to differences in region, terrain, infrastructure, or other factors. Regions with higher average demand lost may indicate more serious reliability issues, while lower demand losses can appear less severe from a system-stress perspective. Still, our data remains incomplete, as evidenced by the missing values.

| NERC Region | Avg Outage Duration (min) | Avg Customers Affected | Avg Demand Loss (MW) |
|:---|---:|---:|---:|
| ASCC        | NaN                        | 14,273.00               | 35.00                 |
| ECAR        | 5,603.31                   | 256,354.19              | 1,314.48              |
| FRCC        | 4,268.73                   | 303,577.14              | 846.79                |
| FRCC, SERC  | 372.00                     | NaN                     | NaN                   |
| HECO        | 895.33                     | 126,728.67              | 466.67                |
| HI          | 1,367.00                   | 294,000.00              | 1,060.00              |
| MRO         | 2,934.95                   | 88,984.97               | 279.50                |
| NPCC        | 3,262.58                   | 108,726.04              | 930.12                |
| PR          | 174.00                     | 62,000.00               | 220.00                |
| RFC         | 3,485.76                   | 128,261.74              | 294.71                |
| SERC        | 1,735.45                   | 107,854.04              | 556.33                |
| SPP         | 2,693.77                   | 188,513.00              | 159.00                |
| TRE         | 2,799.27                   | 226,468.65              | 635.62                |
| WECC        | 1,371.53                   | 133,833.07              | 498.15                |

I also grouped outage counts on state and cause category to see which causes attributed the most to a state's outages. The first five rows are depicted below.

| State | Equipment Failure | Fuel Supply Emergency | Intentional Attack | Islanding | Public Appeal | Severe Weather | System Operability Disruption |
|:---|---:|---:|---:|---:|---:|---:|---:|
| Alabama | 0 | 0 | 1 | 0 | 0 | 4 | 0 |
| Alaska | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| Arizona | 4 | 0 | 18 | 0 | 0 | 3 | 2 |
| Arkansas | 1 | 0 | 6 | 1 | 7 | 10 | 0 |
| California | 21 | 17 | 24 | 28 | 9 | 69 | 41 |

---

## Assessment of Missingness

### Overall Missingness Table

| Column                     | Missing Count |
|:---|:---|
| demand_loss_mw             | 700            |
| customers_affected         | 437            |
| outage_duration            | 58             |
| outage_restoration_datetime | 58            |
| total_sales                | 22             |
| total_price                | 22             |
| month                      | 9              |
| anomaly_level              | 9              |
| climate_category           | 9              |
| outage_start_datetime      | 9              |
| climate_region             | 6              |

### NMAR Analysis
Although several columns contain missing data, I believe that demand_loss_mw is plausibly NMAR, because the probability that this field is missing may depend on the true magnitude of the demand loss itself. For instance, utility companies may not report demand lost if an event is under a certain threshold (systematically excluding small losses in demand), or large, extreme events making it impossible to accurately measure due to how large the demand lost is.

Additional data that could help determine whether demand_loss_mw is MAR could include reporting thresholds (specific cutoffs determined by utility companies on when to report), observed events being heavily linked to demand lost (perhaps certain cause_categories), time data is missing (needed to calculate demand lost), or other factors.

### Missingness Dependency
To test missingness dependancy, I will focus on demand_loss_mw. I will test this against the columns cause_category and month.

#### Cause Category
First, we examine the missingness rates in demand_loss_mw by cause_category.

Null Hypothesis: The missingness of demand_loss_mw does not depend on cause_category.
Alternative Hypothesis: The missingness of demand_loss_mw does depend on cause_category.
<iframe src="https://jyim1203.github.io/power-outages-analysis/https://jyim1203.github.io/power-outages-analysis/plots/missingness_plot.html" width=800 height=600 frameBorder=0></iframe>

I found an observed test statistic of 0.799, meaning the largest difference in missingness rates of demand_loss_mw between any two cause categories was approximately 80%. This corresponded to a p-value of 0.0, which allows us to reject the null hypothesis in favor of the alternative hypothesis. This means that there is a significant difference in the missingness of demand_loss_mw depending on the cause_category, meaning that some cause_category values are much more likely to have missing demand_loss_mw than others.

When compared to the permutation distribution, this observed difference was far larger than what would be expected under the null hypothesis of independence, providing strong evidence that the missingness of demand_loss_mw depends on cause_category.
<iframe src="https://jyim1203.github.io/power-outages-analysis/https://jyim1203.github.io/power-outages-analysis/plots/permutation_test_plot.html" width=800 height=600 frameBorder=0></iframe>

#### Month
Next, we examine the missingness rates in demand_loss_mw by month.

Null Hypothesis: The missingness of demand_loss_mw does not depend on month.
Alternative Hypothesis: The missingness of demand_loss_mw does depend on month.
<iframe src="https://jyim1203.github.io/power-outages-analysis/https://jyim1203.github.io/power-outages-analysis/plots/missingness_by_month.html" width=800 height=600 frameBorder=0></iframe>

I found an observed test statistic of 0.224, meaning the largest difference in missingness rates of demand_loss_mw between any two months was approximately 24%. This corresponded to a p-value of 0.0974, which means we fail to reject the null hypothesis in favor of the alternative hypothesis. There is not a significant difference in the missingness of demand_loss_mw depending on the month, so certain month values are not more likely to have missing demand_loss_mw than others.
<iframe src="https://jyim1203.github.io/power-outages-analysis/https://jyim1203.github.io/power-outages-analysis/plots/permutation_test_by_month.html" width=800 height=600 frameBorder=0></iframe>

---

## Hypothesis Testing

I will be testing whether outages caused by severe weather have longer average durations (compared to other causes). The relevant columns of interest include outage_duration and cause_category.

Null Hypothesis: Outages caused by Severe Weather do not have a statistically significantly longer average duration than outages caused by other sources (mean duration of severe weather outages is the same as mean duration of outages of other sources)

Alternative Hypothesis: Outages caused by Severe Weather have a statistically significantly longer average duration than outages caused by other sources (mean duration of severe weather outages is greater than mean duration of outages of other sources).

Test Statistic: For this test, I used a difference in means (mean log_outage_duration of 'severe weather' outage causes - mean log_outage_duration of all other sources)
- Observed Mean Log Duration (Severe Weather): 7.447
- Observed Mean Log Duration (Other Sources): 4.894
- Observed Difference in Means (Test Statistic): 2.553

I performed a permutation test with 5,000 simulations to generate an empirical distribution of the test statistic under the null hypothesis. 

I got a P-Value of 0.000, so with a standard significance level of 0.05, we reject the null hypothesis in favor of the alternative because the results are statistically significant. On average, the duration of outages caused by severe weather are longer than the durations of outages not caused by severe weather. 

The plot below illustrates our observed difference compared to the empirical distribution of differences under the null generated from the permutation test.
<iframe src="https://jyim1203.github.io/power-outages-analysis/https://jyim1203.github.io/power-outages-analysis/plots/hypothesis_test.html" width=800 height=600 frameBorder=0></iframe>

---

## Framing a Prediction Problem
My model will try to predict the log outage duration (since the normal outage_duration has a heavy right skew) without knowing the exact cause of the outage. This is a regression problem because we want to predict a continuous outcome (log_outage_duration in this case).

The metrics I will be mainly focusing on include mainly R^2 and RMSE to attempt to get as accurate predictions as possible.

At the time of prediction, we would have most features available, including
### Numeric Features
- total_price
- total_sales
- total_customers
- poppct_urban
- popden_urban
- areapct_urban
- util_realgsp
- util_contri
- pc_realgsp_state
- total_realgsp
- population

### Categorical Features
- month
- us_state
- nerc_region
- climate_region
- climate_category
- anomaly_level
- demand_missing
- start_hour
- year

### Excluded Columns
- log_outage_duration
- outage_duration
- outage_restoration_datetime
- demand_loss_mw
- outage_start_datetime
- cause_category
- customers_affected

in our baseline model. This will hopefully allow us to predict how long an outage would last, including some information that might be available to typical consumers while withholding certain other information more involved with the specific outage (such as cause_category, time information, customers_affected, and others that may not be known at the time of the outage).

---

## Baseline Model
My base model is a Ridge Regression model using the features specified above. This hopefully would help provide both individuals affected by the outages more information on what steps they should take to better adapt to the outage while also allowing for emergency services and responders to help optimize response efforts to address these outages.

First, I filtered out observations that were missing values in their log_outage_duration columns, then filtered out observations with 0 customers affected (because customers wouldn't know if an outage occurred if they weren't affected). Then, I split my data into a train (80%) and test (20%) set, using a random state to ensure reproducibility. Next, I fit a pipeline where the missing numerical values were imputed with the median of their columns, while also adding an indicator that these values were imputed. Numerical values were also scaled with StandardScalar, then I did One-Hot Encoding for categorical variables before dropping the excluded columns. 

The performance of this baseline model was quite bad, with an R^2 of 0.2194 and RMSE of 2.0432. This means that (when converted back into minutes) the real performance was off by a Mean Absolute Error of 2664.17 mins and Median Absolute Error of 936.79 mins.

---

## Final Model
My final model is a Random Forest Regressor, which included many of the same features, with some notable differences being that year was removed from the categorical columns and put into the excluded features. Instead, I created 4 new features: 
1. is_weekend (Binary Feature)
   - Derived from: outage_start_datetime.dayofweek in [5, 6]
   - Rationale: Repair crew availability differs on weekends vs weekdays.
     Weekend staffing is typically reduced, potentially leading to longer
     restoration times. This captures day-of-week patterns not present in
     the 'start_hour' feature.

2. year_month (Categorical Feature)  
   - Derived from: Combining year + month as "YYYY-MM"
   - Rationale: Infrastructure quality, repair technology, and utility 
     preparedness improve over time in seasonal patterns. A combined 
     year-month feature captures temporal trends that separate year and 
     month features cannot represent (e.g., winter 2015 vs winter 2010).

3. is_peak_hour (Binary Feature)
   - Derived from: start_hour in [7, 8, 9, 17, 18, 19]
   - Rationale: Outages during peak demand hours (morning/evening commute)
     may receive different prioritization or face different complexity due
     to system strain. Creates a meaningful threshold from continuous hour.

4. econ_stress_region (Binary Feature)
   - Derived from: util_contri > median(util_contri)
   - Rationale: Regions where utilities contribute disproportionately to
     the economy may have different infrastructure investment patterns or
     repair prioritization policies.

Next, I proceeded to use the same imputation, scaling, and OHE pipeline before using GridSearchCV to tune my hyperparameters, getting the best results with
- regressor__max_features: 0.5
- regressor__min_samples_leaf: 4
- regressor__min_samples_split: 10
- regressor__n_estimators: 200

My final metrics included a Test R^2 of 0.3736 (improved by about 15%) and a Test RMSE of 1.8303. In minutes, this new model had Mean Absolute Error of 2096.56 mins and a Median Absolute Error of 723.80 mins, down from by about 567.61 minutes for the MAE and 212.99 for the MAE. Overall, the Random Forest Regressor model outperformed the Ridge Regression model, capturing more non-linear relationships and interactions between features while also fitting patterns better.

---

## Fairness Analysis
My groups for the fairness analysis include Urban and Rural areas, defining areas with above the median population density in urban areas


