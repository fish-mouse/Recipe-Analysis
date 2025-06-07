# Exploring Relationship Between Cooking Time and Average Rating of Recipes with Secondary Focus on Calories Prediction

Author: Ricky Zhang

## Overview
Under the great tutelage and guidance of HDSI Faculties, this project seeks to uncover the relationship between cooking time and average rating and predict calories of the recipes.

## Introduction
Recipes is the verbal vehicle to culinary wisdom passed down from great chefs and to the most humble parents and relatives. [Food.com](https://www.food.com/) serves as one popular website that displays and stores those recipes for the passionate food lovers, struggling college students and adults, and everyday individuals. However, in the age of big data and decling retention rate, relevance are dictated by user experience and presentation. In the context of recipes on [Food.com](https://www.food.com/), elements such as cooking time can heavily impact ratings of recipes, contributing to what recipes gain the most attention. Additionally, predicting calories and verifying the veracity of the reported calories help users make more informed decisions based on dietary goals, in turn may affect a recipe's perceived value. My goal in this report is to **explore the relationship between cooking time** and average rating and predict calories of recipes. I start by analyzing the given two datasets scraped from [Food.com](https://www.food.com/) from the start of 2008 to end of 2018. 


### What is `recipe`
Starting off, `recipe` describes general information about the recipes and the metadata about the submitter. There are a total of 83,782 rows/recipes and 12 columns. Let's try and understand each of the 12 columns!

- `name` (object): the name of the recipe, provided by the user
- `id` (int64): the unique identification of the recipe
- `minutes` (int64): the time it takes to cook/make the meal/item described by recipe, provided by the user
- `contributor_id` (int64): the unique identification of the user who submitted the recipe
- `submitted` (object): the date, month, and year the recipe was uploaded onto Food.com
- `tags` (object): a list-like string describing the tags of the recipe, uploaded by the user
- `nutrition` (object): a list-like string describing the nutrition info of calories, total fat, sugar, sodium, etc, provided by the user
- `n_steps` (int64): the number of steps to complete the recipe, provided by the user
- `steps` (object): string instructions for completing the recipe, provided by the user
- `description` (object): a string text describing the recipe and background information related to the recipe, provided by user
- `ingredients` (object): the str text describing the quantity and the item required to prepare for the recipe, provided by the user
- `n_ingredients` (int64): the number of items required to prepare for the recipe, provided by the user

### What is `ratings`
For the following dataset, `ratings` describes information about their rating, comments, and the metadata about the submitter. There are a total of 731,927 rows/comments and 5 columns. Let's try and understand each of the 5 columns!

- `user_id` (int64): the unique identification of the user who submitted the review
- `recipe_id` (int64): the unique identification of the recipe
- `date` (object): the date, month, and year the comment was uploaded onto a recipe on Food.com
- `rating` (int64): a int evaulation of the recipe (0 to 5), provided by user
- `review` (object): a str text describing user's thoughts, feedbacks, etc to the recipe, provided by user

## Data Cleaning and Exploratory Data Analysis

The datasets are not ready yet. Before making changes, let us see how they look right now (shortened, without ingredients, steps, tags, etc). 

| name                                 |     id |   minutes |   contributor_id | submitted   |   n_ingredients |
|:-------------------------------------|-------:|----------:|-----------------:|:------------|----------------:|
| 1 brownies in the world    best ever | 333281 |        40 |           985201 | 2008-10-27  |               9 |
| 1 in canada chocolate chip cookies   | 453467 |        45 |          1848091 | 2011-04-11  |              11 |
| 412 broccoli casserole               | 306168 |        40 |            50969 | 2008-05-30  |               9 |

Let's see also ratings at its current form (shortened, without review).

|   user_id |   recipe_id | date       |   rating |
|----------:|------------:|:-----------|---------:|
|   1293707 |       40893 | 2011-12-21 |        5 |
|    126440 |       85009 | 2010-02-27 |        5 |
|     57222 |       85009 | 2011-10-01 |        5 |

### Data Cleaning	

#### Cleaning Before Merging
As mentioned previously, columns, `nutrition`, `tags`, `steps`, are list-like strings. I thus parsed them into list of strings or floats, except `nutrition`. For `nutrition`, I broken the column into 7 columns, `calories`, `total fat (PDV)`, `sugar (PDV)`, `sodium (PDV)`, `protein (PDV)`, `saturated fat (PDV)`, and `carbohydrates (PDV)`, for later use. On a minor note, I converted `submitted`, originally data type 'obj', to 'Timestamp'.

In case there n/a placeholders not properly removed, I checked for the presence of common placeholders, such as 'none', 'null', -1, 999, etc. When found, I simply set them as np.nan. During the process, I also inspected the columns with np.nan and questioned the logic. For instance, recipes had 1 name that is np.nan. Given the frequency, it is likely parsing error and therefore needs no further attention. I also found `description` to have 71 as np.nan, but given context, it is reasonable for some users to note include descriptions. By inspecting, I can determine if values needed to be imputed.

In addition, I checked if certain entries should have 0. For instance, there are 26 rows of `calories` that are 0. While some made no sense (likely due to lack of information by the author), some items such as 'detergent' or 'cleaning solution' logically have a calories of 0. There is unfortunately no comprehensive way to impute calories (or other nutritional values) as it requires in depth text analysis, which is outside the scope of this project.

In addition (again), I checked if certain entries should have repeats. For instance, I found `name` to have some duplicates, but given that they can have variations in methodology and ingredients, it makes sense. Of course, for columns such as `id`, there is no repeats as each recipe id is unique. For other columns, we can expect certain amount of repeats!

Now that I'm done with `recipes`, let us inspect `ratings` closely. Let's start off by seeing if there are wacky values afoot. Ratings appear to range from 0 to 5 - 0 being unusual. Upon closer inspection, it appears that users are using reviews to pose questions, thoughts, and reviews of the recipe (given they made tweaks that deviated from the recipes). Therefore, when we later merge the two datasets, we must set them as np.nan since they're invalid ratings! 

In this scenario, I do not need to worry about repeats as it is common to have similar comments and `ratings` made by frequent reviewers. As for potential NA value placeholders, using similar methodology in `recipes`, I found none.

#### Merging the Dataset

Here is how I merged `recipes` and `ratings`
1. Paired all relevant interactions to respective recipes by left merging `recipe` and `ratings`
2. Filled all ratings of 0 with np.nan
3. Grouped all ratings of recipes by average
4. Dropped `review` column

Now that is clean and merged, let's see the result (removed a few column for display). 

| name                                 |     id |   minutes |   n_ingredients |   calories |   average rating |
|:-------------------------------------|-------:|----------:|----------------:|-----------:|-----------------:|
| 1 brownies in the world    best ever | 333281 |        40 |               9 |      138.4 |                4 |
| 1 in canada chocolate chip cookies   | 453467 |        45 |              11 |      595.1 |                5 |
| 412 broccoli casserole               | 306168 |        40 |               9 |      194.8 |                5 |

Onto analysis!

### Univariate Analysis

For univariate analysis, I will observe the distribution of cooking minutes in a recipe. Note that I zoomed in to focus on the distribution under 300 minuutes (or 5 hours) - as initial results goes onward to 1 million minutes. This clearly suggest a significiant positive skew. In addition, most of the distirbution though suggests cooking time to take anywhere from 20 to 65 minutes.

<iframe
  src="assets\Distribution_of_cooking_time.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>

### Bivariate Analysis

For bivariate analysis, I will observe the distirbution of average ratings across binned minutes. I first trimmed the dataset by the 95th percentile of `time` to better reflect a more realistic representation of the cooking time. This value is selected in consideration that 95th percentile which is one of the values closest to multiples of 15. 15 here represent the length of the interval and an arbitrary time difference noticeable when cooking. After trimming, I binned the time by intervals of 15, leading to a total of 17 groups. 

From the observation, we can see a similar instance where average ratings are in the range of 4.5 to 5 across different bins of time, with more ratings in the lower time bins. I would later explore in particular whether high ratings are in fact uniformatlly common across time bins or not.

<iframe
  src="assets\Cooking_Time_Binned_(min)_vs._Average_Ratings.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>

### Interesting Aggregates

Here, I will observe if other variables influence average ratings across time bins, specific to first time user or not and average ratings higher than or equal 4.5. 

| binned minutes   |    False |     True |
|:-----------------|---------:|---------:|
| [0, 15)          | 0.677721 | 0.798081 |
| [105, 120)       | 0.766355 | 0.745495 |
| [120, 135)       | 0.737589 | 0.76055  |
| [135, 150)       | 0.679688 | 0.753056 |
| [15, 30)         | 0.683398 | 0.761646 |
| [150, 165)       | 0.724138 | 0.745353 |
| [165, 180)       | 0.714286 | 0.781395 |
| [180, 195)       | 0.692308 | 0.77305  |
| [195, 210)       | 0.775862 | 0.769412 |
| [210, 225)       | 0.810811 | 0.808511 |
| [225, 240)       | 0.928571 | 0.77907  |
| [240, 255)       | 0.685185 | 0.667519 |
| [30, 45)         | 0.677839 | 0.751912 |
| [45, 60)         | 0.69292  | 0.746869 |
| [60, 75)         | 0.68688  | 0.748744 |
| [75, 90)         | 0.708595 | 0.758336 |
| [90, 105)        | 0.722846 | 0.754039 |

Specifically, I created an additional boolean column, where true is if this is the contributor's first and only time. I determined this by seeing if their `contributor_id` appears once or more than once - in layman's term, whether they submitted more than 1 recipe or not. To note, even though food.com was launched in 1999, it is possible some user who are noted as one time contributor have contributed before 2008. However, within a 10 year timeframe, we can safely consider them as one time contributor if they contributed only once within 10 years. Finally, I use this new column to group the average ratings across time bins 2-way. 

The result shows that first-time contributors have a higher proportion of recipes with high average ratings (>= 4.5), but this may be influenced by lower rating volume per recipe, not necessarily better quality. Therefore, there isn't conclusive evidence or trends to suggest that first-time contributors get more exposure.


## Assessment of Missingness

### NMAR Analaysis

There are three columns with NA values: `name`, `description`, `average rating`. I believe the column `description` is Not Missing At Random (NMAR). To recall, NMAR suggests the missingness of the data in question is dependent on itself. In this case, the presence of `description` logically relies on whether the author believes they have anything to add. Some may believe they must cite the source of where they obtained the recipe, or elaborate that their recipe is modified, or boast how delicious it is. However, either emotions or only obligations determine whether they submit a description. Thus, we can conclude that the missingness of `description` is NMAR.

### Missingness Dependency
For this section, I'll select `average rating` as the column of non-trivial missingness to analyze. To choose the most suited columns and test statisitc to test for dependency, I will plot a few KDE plots comparing the distributions of column with `average rating` missing and not missing. 

Starting off, I selected `n_steps` as the 1st column. I grouped the dataset into `average rating` missing and not missing. I then graphed the KDE of the two dataset under `n_steps`. From the two distributions, we see that while the means roughly align, the peaks have differences. Thus, it is likely that the missingness of `average rating` depends on `n_steps` and that K-S stat is more suitable than mean-based.

<iframe
  src="assets\KDE_NStep_vs_AvgRatingMissing.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>

Next, I selected `n_ingredients` as the 2nd column. I grouped the dataset into `average rating` missing and not missing. I then graphed the KDE of the two dataset under `n_ingredients`. From the two distributions, we see that while the means roughly align, the peaks have little differences. Thus, it is likely that the missingness of `average rating` does not depends on `n_steps` and that K-S stat is more suitable than mean-based.

<iframe
  src="assets\KDE_NIngred_vs_AvgRatingMissing.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>

To confirm my prior conjecture, I will perform permutation testing. Firstly, let us focus on `n_steps`.

<br>**Significance Level:** 0.01
<br>**Test Statisitc:** K-S Statistic
<br>**Null Hypothesis:** the missingness of `average rating` does not depend on `n_steps` in the recipe.
<br>**Alternate Hypothesis:** the missingness of `average rating` does depend on `n_steps` in the recipe.

<iframe
  src="assets\NStepsAvgRatingMissingNull.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>

As shown in the plot above and the p_val calculation below, we see 0.01 > p_val = 0.00. Thus, we reject the null hypothesis. As a result, the missingness of `average rating` does depend on `n_steps`.

Finally, let us focus on `n_ingredients`.

<br>**Significance Level:** 0.01
<br>**Test Statisitc:** K-S Statistic
<br>**Null Hypothesis:** the missingness of `average rating` does not depend on `n_ingredients` in the recipe.
<br>**Alternate Hypothesis:** the missingness of `average rating` likely does depend on `n_ingredients` in the recipe.

<iframe
  src="assets\NIngredAvgRatingMissingNull.html"
  width="700"
  height="450"
  frameborder="0"
></iframe>

As shown in the plot above and the p_val calculation, we see 0.01 < p_val = 0.026. Thus, we fail to reject the null hypothesis. As a result, the missingness of `average rating` likely does not depend on `n_ingredients`.

## Hypothesis Testing

In the interest of determining the relationship between minutes and average rating, I focused on whether cooking time is associated with a high average rating (defined as ≥ 4.5). To answer this, I conducted a permutation test.

<br>**Test Statistic**: The sum of squared differences between the observed and expected proportions of high-rated recipes across cooking time bins.
<br>**Significance Level**: 0.01
<br>**Null Hypothesis (H₀)**: The proportion of highly rated recipes is the same across all cooking time bins.
<br>**Alternative Hypothesis (H₁)**: The proportion of highly rated recipes differs in at least one cooking time bin.

To construct the test, I trimmed minutes at the 95th percentile to prevent extreme values from skewing the analysis, then grouped cooking time into bins of 15 minutes. Under the null hypothesis, we expect the proportion of high-rated recipes (≥ 4.5) to be uniformly distributed across the 17 bins — approximately 1/17 per bin. To measure deviation, I calculated the squared differences between the actual and expected proportions, summing across all bins. This penalizes larger deviations and ensures all differences are positive. Empty bins from sampling were assigned the full squared difference of the expected value.

I sampled 2000 recipes per simulation and shuffled the “Above 4.5” labels 10,000 times to generate a null distribution.

P-value: 0.0001

Since 0.0001 < 0.01, we reject the null hypothesis. This suggests that the proportion of highly rated recipes is likely not uniform across cooking time bins — indicating a possible association between cooking time and high average ratings.

## Framing a Prediction Problem

In the current and following section, I will focus on predicting calories of the recipe in a regression problem approach. I chose calories as the response variable because this is one of the primary reasons for users in selecting which recipes to cook. I will be using RMSE as my metric for the following reasons. Firstly, I'm prioritizing how off I'm from the correct value. Secondly, I do not want to focus on capturing the overall variance of the dataset as the dataset has skewed data and noise (due to the encompassing of non-food recipes and self-reported nutritional values that may be inaccurate).

Lastly, the relevant information I know at the time of prediction are `tags`, `minutes`, `n_steps`, `steps`, `n_ingredients`,  `ingredients`, and `average rating`. I will both derive and modify select features listed to train the model.

## Baseline Model

### Features

In this section, to build the baseline model, I refamiliarize with the distribution of the columns. `Calories`, `n_steps` and `minuest` (in particular) have strong positive skewnees. Therefore, I transformed Calories logarithmically to reduce the skewness. Following, I select and modified the features as such. Firstly, I log transformed minutes using `FunctionTransformer` and `np.log1p`. Then I created an interactive term by multiplying `n_steps` and `n_ingredients`. The rationale is that the more steps and ingredients there is, the more processed potentially the food becomes, and thus higher calories.  In total, I have 2 quantitative features.

### Performance & Model

I then splitted the dataset into 80% training data and 20% test data. Instead of simple linear regression, I used Ridge, a regularized model, because in the final model, I plan to reuse `n_ingredients` for another feature, which may introduce instability and multicollinearity. 

After fitting the data, I asked the model to predict both the training and test features. The RMSE from training is around 0.890, while the RMSE from testing is around 0.889. This suggests that the baseline model generalizes well to unseen data and is not overfitting, since the training and testing errors are nearly identical.

## Final Model

### New Features

In this section, to build the final model, I added/revised the following features. In ordered to better capture nonlinear trends, I incorporate the `FunctionTransformer` for the interactive term within a pipeline that further transforms it using `PolynomialFeatures` I currently left it as degree = 1 as default and will explore potential improvement while tuning with hyperparameters. 

Regarding the two new numeric features I implemented, I built two custom Transformers with `BaseEstimator` and `TransformerMixin`: `TopIngredientsRatioEncoder` and `TopTagsRatioEncoder`. Both follow similar logic. Then, for each recipe, they compute the ratio of how many of its ingredients or tags are in the top 20 set relative to the total number of ingredients or tags it contains.

Starting off, I extracted the top 20 most frequent strings from `ingredients` and `tags` of the entire dataset to identify the most typical or popular ingredients and tags used. Instead of keep them as raw counts, I converted them into ratios to normalize across recipes of different lengths. This helps avoid bias where recipes with more ingredients or tags could score higher due to size. I believe these ratios are meaningful because they capture how much a recipe aligns with common culinary patterns, which may influence calorie content indirectly through standardization or popularity of preparation styles.

### Model & Hyerparameters

As mentioned earlier, I used Ridge rather than simple linear regression model to lessen multicollinearity introduced by reusing `n_ingredients`.

On hyperparameters, I used three: `ridge__alpha`, `columntransformer__transformer_weights`, and `columntransformer__n_ingred_log_step_poly__poly__degree`. To explain, since I'm using Ridge as my model, I can tune the `alpha` parameter to control how strongly I penalize large coefficients - resulted by multi-collinearity. For the second hyperparameter, I control what features I have on. Due to compile time and resource limitation, I choose 3 combinations: all features on, baseline features on, and new final model features on. Lastly, I ask the Best Hyperparameter Seasrch Algorithm to tune `PolynomialFeatures` with degree of 1 to 3. This allows me to capture nonlinear trend without underfitting. 

Given the limited combinations, I used `GridSearchCV` to iteratively find the best combination of hyperparameters while applying 5-fold cross-validation. As a result, the best hyperparameters are:
1. Degree of 3 for `n_ingred_log_step_poly`
2. All features of the final model
3. Ridghe__alpha of 10

### Performance

From the final model, I obtained 0.886 from the train RMSE and 0.884 from the test RMSE. Sefl-contained, this suggests that the final model generalizes well to unseen data and is not overfitting, since the training and testing errors are nearly identical.

In comparison with the baseline model's performance, there is a 0.6% decrease in RMSE, suggesting an improvement.

## Fairness Analysis

In this final seciton, I performed a fairness analysis using permutation test on two groups: recipes with **long** cooking time and recipes with **short** cooking time. In this context, I define long cooking time as cooking time less than the median of `minutes`. 

<br>**Null Hypothesis:** Our model is fair. Its errir for long and short cooking time recipes are roughly the same, and any differences are due to random chance.
<br>**Alternate Hypothesis:** Our model is unfair. Its error for shorter cooking time recipes is worse than its error for longer cooking time recipes. 
<br>**Significance Level**: 0.01
<br>**Evaulation Metric**: RMSE

<br>**P_val**: 0.0

After performing the permutation test, I obtained a P_val of 0.0, which is smaller than our significance level. As a result, we reject the null hypothesis. This suggests that my model likely performs better on calories perdiction for recipes with higher cooking times, but worse on recipes with lower cooking times. This potentially is due to utilization of features that are more representative of calories of recipes with longer cooking time. Overall, the model shows bias toward one group, indicating an area for improvement in fairness.