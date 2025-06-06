# Exploring Relationship Between Cooking Time and Average Rating of Recipes with Secondary Focus on Calories Prediction

Authors: Ricky Zhang

## Overview
Under the great tutelage and guidance of HDSI Faculties, this project seeks to uncover the relationship between cooking time and average rating and predict calories of the recipes.

## Introduction
Recipes is the verbal vehicle to culinary wisdom passed down from great chefs and to the most humble parents and relatives. [Food.com](https://www.food.com/) serves as one popular website that displays and stores those recipes for the passionate food lovers, struggling college students and adults, and everyday individuals. However, in the age of big data and decling retention rate, relevance are dictated by user experience and presentation. In the context of recipes on [Food.com](https://www.food.com/), elements such as cooking time can heavily impact ratings of recipes, contributing to what recipes gain the most attention. Additionally, predicting calories and verifying the veracity of the reported calories help users make more informed decisions based on dietary goals, in turn may affect a recipe's perceived value. My goal in this report is to explore the relationship between cooking time and average rating and predict calories of recipes. I start by analyzing the given two datasets scraped from [Food.com](https://www.food.com/) from the start of 2008 to end of 2018. 


## What is `recipe`
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

## What is `ratings`
For the following dataset, `ratings` describes information about their rating, comments, and the metadata about the submitter. There are a total of 731,927 rows/comments and 5 columns. Let's try and understand each of the 5 columns!

- `user_id` (int64): the unique identification of the user who submitted the review
- `recipe_id` (int64): the unique identification of the recipe
- `date` (object): the date, month, and year the comment was uploaded onto a recipe on Food.com
- `rating` (int64): a int evaulation of the recipe (0 to 5), provided by user
- `review` (object): a str text describing user's thoughts, feedbacks, etc to the recipe, provided by user
