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

<iframe
  src="assets\Distribution_of_cooking_time.html"
  width="700"
  height="500"
  frameborder="0"
></iframe>


## Bivariate Analysis


## Interesting Aggregates