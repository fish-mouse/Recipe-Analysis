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

As shown in the plot above and the p_val calculation below, we see 0.01 > p_val = 0.0.0. Thus, we reject the null hypothesis. As a result, the missingness of `average rating` does depend on `n_steps`.

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

Test Statistic: The sum of squared differences between the observed and expected proportions of high-rated recipes across cooking time bins.

Significance Level: 0.01
Null Hypothesis (H₀): The proportion of highly rated recipes is the same across all cooking time bins.
Alternative Hypothesis (H₁): The proportion of highly rated recipes differs in at least one cooking time bin.

To construct the test, I trimmed minutes at the 95th percentile to prevent extreme values from skewing the analysis, then grouped cooking time into bins of 15 minutes. Under the null hypothesis, we expect the proportion of high-rated recipes (≥ 4.5) to be uniformly distributed across the 17 bins — approximately 1/17 per bin. To measure deviation, I calculated the squared differences between the actual and expected proportions, summing across all bins. This penalizes larger deviations and ensures all differences are positive. Empty bins from sampling were assigned the full squared difference of the expected value.

I sampled 2000 recipes per simulation and shuffled the “Above 4.5” labels 10,000 times to generate a null distribution.

P-value: 0.0001

Since 0.0001 < 0.01, we reject the null hypothesis. This suggests that the proportion of highly rated recipes is likely not uniform across cooking time bins — indicating a possible association between cooking time and high average ratings. 

## Framing a Prediction Problem

### Problem Identification	

## Baseline Model

### Baseline Model	


## Final Model

### Final Model	


## Fairness Analysis

### Fairness Analysis	