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

| columns        |missing|
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

- For the interested variable `'easy'`, it has 136,438 `True`s and 97,991 `False`s. Distribution of `'easy'`:

<iframe src="assets/easy_barh.html" width=450 height=450 frameBorder=0></iframe>

Let's observe each column of the descriptive statistics table and take a look at their distributions. 

- We can observe there're outliers in the distribution of `'minutes'`. The difference betweeen maximum (over 1 million minutes?!) and third quartile (65 min) is too big, and the minimum is 0 minutes (this can't happen!). So when plot it, I will filter out the outliers and only use the middle 98% of data. 

Distribution of `'minutes'`:

<iframe src="assets/mins_hist.html" width=600 height=450 frameBorder=0></iframe>

It tells that most of recipes can be finished in one and a half hour, but some of them can take really long time.

- Then take a look at `'calories'`. It also has outliers due to similar reason (find this by observing the descriptive statistics), so we still use the middle 98% of data.

Distribution of `'calories'`:

<iframe src="assets/cals_hist.html" width=600 height=450 frameBorder=0></iframe>

The distribution of `'calories'` is very right-skewed and relatively smooth. Its center is around 200 Cal, but there're also some recipes which exceed 1500 Cal!

- `'n_steps'` have normal minimum and abnormal maximum, so just filter out big outliers - we'll use the first 98% of data and exclude the biggest 2%. 

Distribution of `'n_steps'`:

<iframe src="assets/nsteps_hist.html" width=600 height=450 frameBorder=0></iframe>

Most recipes have no more than 20 steps. The distribution is overall smooth and a little bit right-skewed. 

- `'n_ingredients'` has a similar situation with `'n_steps'`, but it should be less skewed as its mean and median (second quartile) is closer than `'n_steps'`'s mean and median. 

Distribution of `'n_ingredients'`:

<iframe src="assets/ningre_hist.html" width=600 height=450 frameBorder=0></iframe>

This distribution is the most normal one among all the univariate analyses!

- The last one is `'avg_rating'`, which has no outliers as it's limited in 1 to 5 exclusively. 

Distribution of `'avg_rating'`:

<iframe src="assets/avgrt_hist.html" width=600 height=450 frameBorder=0></iframe>

We can see most of the average ratings are relatively high, larger than 3.5 or even 4. 


#### Bivariate Analysis & Aggregates

First I'd like to show a scatterplot matrix to give a general impression of relationships between variables. 

<iframe src="assets/scatmat.html" width=600 height=450 frameBorder=0></iframe>

Unfortunately, I have to say I don't find any obvious, strong relationship. 

So for the next step, let's use tables and conditional histograms to find possible differences by `'easy'`.

| easy   |   ('id', 'mean') |   ('id', 'median') |   ('minutes', 'mean') |   ('minutes', 'median') |   ('n_steps', 'mean') |   ('n_steps', 'median') |   ('n_ingredients', 'mean') |   ('n_ingredients', 'median') |   ('rating', 'mean') |   ('rating', 'median') |   ('avg_rating', 'mean') |   ('avg_rating', 'median') |   ('calories', 'mean') |   ('calories', 'median') |
|:-------|-----------------:|-------------------:|----------------------:|------------------------:|----------------------:|------------------------:|----------------------------:|------------------------------:|---------------------:|-----------------------:|-------------------------:|---------------------------:|-----------------------:|-------------------------:|
| False  |           381963 |             378410 |               94.4948 |                      45 |               12.6056 |                      11 |                    10.7953  |                            10 |              4.67391 |                      5 |                  4.41879 |                    4.66667 |                481.768 |                   354.2  |
| True   |           366845 |             353171 |              115.62   |                      30 |                8.1593 |                       7 |                     7.83344 |                             7 |              4.68412 |                      5 |                  4.44181 |                    4.66667 |                374.828 |                   268.75 |

By comparing the means and medians across two groups, it seems that: among the five variables analyzed univariately above, the four columns except `'avg_rating'` (and `'rating'`) have some kinds of differences across easy or not. Let's look more by graphing.

Below I will show four histograms which show the distributions of the four variables grouping by the binary variable `'easy'`. I will give the analysis after all the four graphs. 

- `'n_ingredients'`, number of ingredients

<iframe src="assets/ningre_easy.html" width=600 height=450 frameBorder=0></iframe>

- `'n_steps'`, number of steps

<iframe src="assets/nsteps_easy.html" width=600 height=450 frameBorder=0></iframe>

- `'calories'`

<iframe src="assets/cals_easy.html" width=600 height=450 frameBorder=0></iframe>

- `'minutes'`

<iframe src="assets/mins_easy.html" width=600 height=450 frameBorder=0></iframe>

Analysis: 
The most obvious differences occur in `'n_steps'` and `'n_ingredients'`, which are very reasonable: recipes with less steps and less materials are definitely easier to complete. But one thing we can observe is the difference in shapes of `'minutes'`, although the two distributions are similar in trends. The difference in `'minutes'` is not that intuitively and thus interesting. 
We can choose `'minutes'` as the variable of intereset to perform the permutation test. 


---


## Assessment of Missingness

We have three columns in the dataset with missing values: `'description'`, `'rating'`, and `'review'`. `'avg_rating'` is calculated by `'rating'`, so we don't consider it as a separate column. As `'description'` and `'review'` are not important due to my topic, I will perform missingness dependncy tests on `'rating'`. 

### NMAR Analysis

Conclusion in short: For the three columns with missingness, I belive that `'description'` is NMAR, and `'rating'` and `'review'` are MAR. 

- All these missingness have a same basic reason: the users don't want to fill them when using the webpage. Sometimes they don't want to write the description. Perhaps they only want to leave a rating with no words to say, and sometimes they want to discuss but not rate. When we scrape the data from webpage, these empty places are recorded as missing. 

- `'description'` is possibly NMAR: I don't think the connection between a description and other variables of a recipe is close and clear - in this dataset, I don't find any column which might affect people's willingness to fill the description. 

- `rating` and `review` are possibly MAR: User's willingness can be affcted much by other factors of the recipe when they are rating or leaving comments. For example, the easiness of cooking, the steps of recipe, or the nutrition, even the taste (as a tag). They appear more obvious relationship with other columns than `'description'`. 



### Missingness Dependency

For this part, I create a binary variable `'rating_missing'` which indicates the corresponding `'rating'` is missing (`True`) or not (`False`).
Let's observe a table of means and medians for `df` grouped by `'rating_missing'`. That is, we grouped the dataset into two groups, one with missingness in `'rating'` and one without. Then we get their means and medians within the group. 

| rating_missing   |   ('minutes', 'median') |   ('minutes', 'mean') |   ('n_steps', 'median') |   ('n_steps', 'mean') |   ('n_ingredients', 'median') |   ('n_ingredients', 'mean') |   ('rating', 'median') |   ('rating', 'mean') |   ('easy', 'median') |   ('easy', 'mean') |   ('calories', 'median') |   ('calories', 'mean') |
|:-----------------|------------------------:|----------------------:|------------------------:|----------------------:|------------------------------:|----------------------------:|-----------------------:|---------------------:|---------------------:|-------------------:|-------------------------:|-----------------------:|
| False            |                      35 |               103.49  |                       9 |               9.93198 |                             9 |                     9.0612  |                      5 |              4.67987 |                    1 |           0.583546 |                   299.4  |                415.103 |
| True             |                      40 |               154.942 |                       9 |              11.2706  |                             9 |                     9.22193 |                    nan |            nan       |                    1 |           0.559457 |                   327.45 |                484.11  |

Observe the median to get rid of the effects of outliers. We can find that `'minutes'` and `'calories'` have different medians across groups, and the rest variables (except `'rating'`) do not. 

Then I will graph kernel density estimates (using `plotly.figure_factory.create_distplot` as we did in lecture) for chosen columns to explore about their relationship with missingness. 

#### `'calories'`: depend on missingness

Let's plot the distribution of `'calories'` by missingness in `'rating'` to take a look at the two distributions across groups. 

<iframe src="assets/cals_miss.html" width=600 height=450 frameBorder=0></iframe>

We can find that the two distributions have similar shapes but different center statistics. In this case, we should use difference in mean/median to perform the permutation test on missingness. As the distributions are skewed by outliers seriously, we'd better choose median to ensure the accuracy. Also, the number of simulations in permutation test is set to be 100. 

After performing the test, we get the **p-value = 0**, smaller than typical significance level 0.05. Thus, we have evidence to support that the distributions of `'calories'` differ across the missingness of `'ratings'`.

#### `'n_ingredients'`: not depend on missingness

Similarly, I plot the distribution of `'n_ingredients'` by missingness in `'rating'`. 

<iframe src="assets/ningre_miss.html" width=600 height=450 frameBorder=0></iframe>

Although the distribution of `'n_ingredients'` when missing rating seems to have an strange fluctuation, but the two distributions are similar in general shapes, and their center statistics are located similarly at about 9. 

After performing the test using difference of medians as the test statistics, we get the **p-value = 1.0**, larger than typical significance level 0.05. That is, we don't have evidence to reject that the distributions of `'n_ingredients` number of ingredients are same when `'rating'` is missing or not - we may accept that they are same. 




## Hypothesis Testing

As mentioned before in the EDA part, the variable I'm interested in is `'minutes'`: I think `'minutes'` may have different distributions by the groups of two values of `'easy'`.

#### Hypothesis

*Null Hypothesis*: The distribution of minutes in recipes when the recipe is easy is same as when the recipe is not easy.

*Alternative Hypothesis*: The two distributions of minutes are different across the two groups.

#### Test Statistics

Let's first observe the descriptive statistics and graph to determine the test statistics we should use. 

I exclude some extreme outliers (like 10,000 minutes) to avoid too much distractions on means. Here's the table of `'minutes'`'s mean and medians within the two groups: 

| easy   |    mean |   median |
|:-------|--------:|---------:|
| False  | 69.3791 |       45 |
| True   | 53.2087 |       30 |

We can see the center statistics are different across groups. Let's look at the graph to get the shapes of whole distributions: 

<iframe src="assets/hypotest.html" width=600 height=450 frameBorder=0></iframe>


I mentioned similar things above, but to recap: first, we are comparing two numeric distributions. Also, by the graph, the two distributions are likely to have different means and medians but similar shapes. So we should use difference in group means or medians as the test statistic. As means are sensitive to outliers and `minutes` have some abnormal outliers, I will use difference in medians as the test statistic.

For the significance level, we can try three values that are usually used in statistics: 0.01, 0.05, 0.1. On one hand, people usually use these significance levels, and we can have a try. On the other hand, this choice can show the possibly different results of rejecting or not rejecting null hypothesis under different tolerance to type-I error, false rejection. 

After permutation test (using the original `'minutes'` column without excluding extreme outliers), we get **p-value = 0.0**. Taking anyone of the three significance level, we should reject the null hypothesis. We have enough evidence to support that the distribution of minutes in recipes when the recipe is easy is **different** to when the recipe is not easy.

In conclusion, recipes that are considered easy should have a short time in minutes to cook in general!

