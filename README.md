# Recipes Analysis: what types of recipes are easy to make? 
Project 3 for DSC 80 by Rosie Peng (q1peng@ucsd.edu).

## Introduction

In this project, we studied what characteristics of a recipe is related to "easy to cook", which is defined as having a "easy" related tag. 

If we can figure out the answer of this question, the contributors will be able to make their recipes easier to make, and more people can try to cook! So cool ;D

#### Introduction of the dataset

The dataset `Recipes and Ratings` contains a lot of data about recipes from food.com, a recipe website. We can see things about the recipes themselves: ID, information about the uploaders, details about the steps and foods. It also includes data about ratings, such as users, rating times, comments, etc. 

- In the data file `recipes`, there're 83,782 rows and 12 columns. 

- In the data file `ratings`, there're 731,927 rows and 5 columns. 

The names and descriptions of relevant columns:

From `recipes`:

| Column | Description |
| --- | --- |
| `id` | Recipe ID |
| `minutes` | Minutes to prepare recipe |
| `tags` | Food.com tags for recipe |
| `nutrition` | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `n_steps` | Number of steps in recipe |


From `ratings`:

| Column | Description |
| --- | --- |
| `recipe_id` | Recipe ID |
| `rating` | Rating given |


 
#### Question

**What types of recipes tend to be easy to make?**

- To be specific, I define "easy to make" as having a tag contains "easy". I will pick the variables of interest on recipes and perform permutation testing.

---

## Cleaning and EDA

##### Package Used

I used `pandas`, `numpy`, and `os` to deal with data. For plotting, I used `plotly.express` and `plotly.figure_factory`. 

### Data Cleaning

First let's take a look and have an impression of the two raw DataFrames `recipes` and `ratings`. 

A row from `recipes` (too long to include more!):

| name                                 |     id |   minutes |   contributor_id | submitted   | tags                                                                                                                                                                                                                        | nutrition                                |   n_steps | steps                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | description                                                                                                                                                                                                                                                          | ingredients                                                                                                                                                                    |   n_ingredients |
|:-------------------------------------|-------:|----------:|-----------------:|:------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------|----------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------:|
| 1 brownies in the world    best ever | 333281 |        40 |           985201 | 2008-10-27  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'for-large-groups', 'desserts', 'lunch', 'snacks', 'cookies-and-brownies', 'chocolate', 'bar-cookies', 'brownies', 'number-of-servings'] | [138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0] |        10 | ['heat the oven to 350f and arrange the rack in the middle', 'line an 8-by-8-inch glass baking dish with aluminum foil', 'combine chocolate and butter in a medium saucepan and cook over medium-low heat , stirring frequently , until evenly melted', 'remove from heat and let cool to room temperature', 'combine eggs , sugar , cocoa powder , vanilla extract , espresso , and salt in a large bowl and briefly stir until just evenly incorporated', 'add cooled chocolate and mix until uniform in color', 'add flour and stir until just incorporated', 'transfer batter to the prepared baking dish', 'bake until a tester inserted in the center of the brownies comes out clean , about 25 to 30 minutes', 'remove from the oven and cool completely before cutting'] | these are the most; chocolatey, moist, rich, dense, fudgy, delicious brownies that you'll ever make.....sereiously! there's no doubt that these will be your fav brownies ever for you can add things to them or make them plain.....either way they're pure heaven! | ['bittersweet chocolate', 'unsalted butter', 'eggs', 'granulated sugar', 'unsweetened cocoa powder', 'vanilla extract', 'brewed espresso', 'kosher salt', 'all-purpose flour'] |               9 |


`ratings`:

|   user_id |   recipe_id | date       |   rating | review                                                                                                                                                                                                        |
|----------:|------------:|:-----------|---------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|   1293707 |       40893 | 2011-12-21 |        5 | So simple, so delicious! Great for chilly fall evening. Should have doubled it ;)<br/><br/>Second time around, forgot the remaining cumin. We usually love cumin, but didn't notice the missing 1/2 teaspoon! |
|    126440 |       85009 | 2010-02-27 |        5 | I made the Mexican topping and took it to bunko.  Everyone loved it.                                                                                                                                          |
|     57222 |       85009 | 2011-10-01 |        5 | Made the cheddar bacon topping, adding a sprinkling of black pepper. Yum!                                                                                                                                     |

*First deals with missingness:*

Below is the missingness situation of each column in the dataframe:

|                |     0 |
|:---------------|------:|
| name           |     1 |
| id             |     0 |
| minutes        |     0 |
| contributor_id |     0 |
| submitted      |     0 |
| tags           |     0 |
| nutrition      |     0 |
| n_steps        |     0 |
| steps          |     0 |
| description    |   114 |
| ingredients    |     0 |
| n_ingredients  |     0 |
| user_id        |     1 |
| recipe_id      |     1 |
| date           |     1 |
| rating         | 15036 |
| review         |    58 |

- After checking, we find the missingness in `'name'`, `'user_id'`, `'recipe_id'`, and `'date'` is due to a small mistake. It's okay to ignore it and do nothing. 

- To work on the ratings of each recipe, we need to combine the two datasets. Also, we know that `'rating'` is actually a number between 1 and 5 inclusively. When we have a 0 for this variable, the thing happens is the user only wrote a review and didn't leave an rating. In this case, we should treat 0 as a missingness. So this step includes merging the two datasets by recipe IDs and filling in zeroes in `'rating'` with `np.nan`.

- For the sake of explotary analysis, we calculated the average rating for each recipe and combined this variable into the dataset as `'avg_rating'`, as this is more representative for the general view on a recipe than single rating. For the same reason said above, when we see 0 in this column, it means nobody rated this recipe. So we replaced 0s in `'avg_rating'` with `np.nan`, too. 

*Type issues:*

- We also fix the types of columns. `'submitted'` and `'date'` should be date objects, but they were objects initially. So we transformed them into datetime objects, although these two columns are not used in this analysis :(

- The contents of columns `'tags'` and `'nutrition'` are strings which have an appearance of list. We convert them into real lists of strings with string methods of `pandas`.

*Create new variables needed:*

- Due to the question, we need to extract the "easy" tag from `'tags'`. Also, we are interested about calories of recipes as a variable, so we need to extract the calories from `'nutrition'`. 

- For this sake, we checked if each recipe has the tag contains the word "easy" and assigned the result to a new binary column `'easy'`, whose value of `True` implies the recipe has a tag which contains "easy". 

- Also, we extracted the first float number from each `'nutrition'` and assigned them to a new float column `'calories'`, indicating the number of calories in the recipes. 

Finally, we got rid of all the variables we didn't need. And the result DataFrame `df` we used for analysis looks like: 

|     id |   minutes |   n_steps |   n_ingredients |   rating |   avg_rating | easy   |   calories |
|-------:|----------:|----------:|----------------:|---------:|-------------:|:-------|-----------:|
| 333281 |        40 |        10 |               9 |        4 |            4 | False  |      138.4 |
| 453467 |        45 |        12 |              11 |        5 |            5 | False  |      595.1 |
| 306168 |        40 |         6 |               9 |        5 |            5 | True   |      194.8 |
| 306168 |        40 |         6 |               9 |        5 |            5 | True   |      194.8 |
| 306168 |        40 |         6 |               9 |        5 |            5 | True   |      194.8 |

It's very tidy now! 

### Exploratory Descriptive Analysis (EDA)

First we checked the distributions of variables with `df.describe()`. 

|       |     id |         minutes |      n_steps |   n_ingredients |        rating |    avg_rating |   calories |
|:------|-------:|----------------:|-------------:|----------------:|--------------:|--------------:|-----------:|
| count | 234429 | 234429          | 234429       |    234429       | 219393        | 231652        | 234429     |
| mean  | 373164 |    106.79       |     10.0178  |         9.07151 |      4.67987  |      4.43221  |    419.529 |
| std   |  67801 |   3285.97       |      6.44226 |         3.82304 |      0.710471 |      0.713305 |    583.224 |
| min   | 275022 |      0          |      1       |         1       |      1        |      0.25     |      0     |
| 25%   | 314272 |     20          |      6       |         6       |      5        |      4        |    170.7   |
| 50%   | 363144 |     35          |      9       |         9       |      5        |      4.66667  |    301.1   |
| 75%   | 424517 |     60          |     13       |        11       |      5        |      5        |    491.1   |
| max   | 537716 |      1.0512e+06 |    100       |        37       |      5        |      5        |  45609     |

#### Univariate Analysis

- For the interested variable `'easy'`, it has 136,438 `True`s and 97,991 `False`s. 

<iframe src="assets\easy_barh.html" width=600 height=450 frameBorder=0></iframe>

- 


## Assessment of Missingness



## Hypothesis Testing

