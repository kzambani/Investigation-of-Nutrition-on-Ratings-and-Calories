# Investigation of Nutrition on Ratings and Calories
By: Karina Zambani and Ivan Chen

## Introduction
In today's diet culture, we are seeing the rapid rise of focus placed on protein. You can't watch a single recipe on TikTok or a gym Youtube video without protein being emphasized for health and fitness. Because of this, we were interested in if **the protein content of a recipe would affect its rating.**

As such, we are exploring two datasets to answer this question. Both are sourced from [food.com](food.com), which is a popular website for recipes where users can leave reviews.

The first dataset is `recipes`, which includes 83782 unique recipes with the following 10 columns:

| Column        | Description                                                                 |
|---------------|-----------------------------------------------------------------------------|
| `name`        | Recipe name                                                                 |
| `id`          | Recipe ID                                                                   |
| `minutes`     | Minutes to prepare recipe                                                   |
| `contributor_id` | User ID who submitted this recipe                                          |
| `submitted`   | Date recipe was submitted                                                   |
| `tags`        | Food.com tags for recipe                                                    |
| `nutrition`   | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `n_steps`     | Number of steps in recipe                                                   |
| `steps`       | Text for recipe steps, in order                                             |
| `description` | User-provided description                                                   |

The second dataset is `interactions`, which includes 731927 user reviews with the following 5 columns:

| Column      | Description                         |
|-------------|-------------------------------------|
| `user_id`   | User ID                             |
| `recipe_id` | Recipe ID                           |
| `date`      | Date of interaction                 |
| `rating`    | Rating given                        |
| `review`    | Review text                         |

This analysis will allow us to determine the weight that people might give protein to when it comes to cooking, and if explored further, give some idea to what kinds of people are more health-conscious.

## Data Cleaning and Exploratory Data Analysis
### Data Cleaning Steps

1. Performed a left merge of the `recipes` and `interactions` datasets on their unique recipe IDs. Each recipe is now matched to its review.
2. Filled any ratings valued "0" with np.nan. Ratings are only offered on a range of 1-5, so a rating of 0 is an indication that the value is actually missing. When performing aggregations later on, we want to avoid any missing values dragging values in a certain direction, so we replaced them with null values.
3. Computed and added a column for `average_rating` of each recipe. This is to compress down the ratings for each recipe into a single value we can work with.
4. There are 3 columns that appear to contain lists of values, but are actually strings. For the `tags`, `nutrition`, and `steps` columns, we converted these values to actual lists of strings and floats. We want to be able to work with individual nutrition values and assess specific tags. 
5. Separated the `nutrition` column list of values into separate columns. This lets us assess specific types of nutrition in our analysis. 
6. Computed and added a column for `prop_protein` which contains the proportion of protein of a recipe that contributes to its total calories. The FDA (Food and Drug Association) state that the recommended amount of protein per day is 50g and that each gram contains 4 calories. We used these numbers to calculate this proportion from the `protein (PDV)` column. Our analysis aims to connect protein to ratings, so we needed a way to evaluate protein content in a standardized way. 

Our cleaned dataset is displayed below. It only shows the first 5 rows, but there are 83782 total. We have also omitted columns not relevant to our analysis, but all of the columns are: [`name`, `id`, `minutes`, `contributor_id`, `submitted`, `tags`, `nutrition`, `n_steps`, `steps`, `description`, `ingredients`, `n_ingredients`, `user_id`, `recipe_id`, `date`, `rating`, `review`, `average_rating`, `calories (#)`, `total fat (PDV)`, `sugar (PDV)`, `sodium (PDV)`, `protein (PDV)`, `saturated fat (PDV)`, `carbohydrates (PDV)`, `prop_protein`, `rating_missing`, `prop_sat_fat`, `n_tags`, `is_high_protein`]


| name                                  | id       | minutes | nutrition                                              | n_steps | n_ingredients | rating | average_rating | calories (#) | total fat (PDV) | sugar (PDV) | sodium (PDV) | protein (PDV) | saturated fat (PDV) | carbohydrates (PDV) | prop_protein |
|---------------------------------------|----------|---------|-------------------------------------------------------|---------|---------------|--------|----------------|--------------|----------------|-------------|--------------|---------------|---------------------|---------------------|--------------|
| 1 brownies in the world best ever     | 333281   | 40      | [138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0]             | 10      | 9             | 4.0    | 4.0            | 138.4        | 10.0           | 50.0        | 3.0          | 3.0           | 19.0                | 6.0                 | 0.04         |
| 1 in canada chocolate chip cookies    | 453467   | 45      | [595.1, 46.0, 211.0, 22.0, 13.0, 51.0, 26.0]         | 12      | 11            | 5.0    | 5.0            | 595.1        | 46.0           | 211.0       | 22.0         | 13.0          | 51.0                | 26.0                | 0.04         |
| 412 broccoli casserole                | 306168   | 40      | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0]            | 6       | 9             | 5.0    | 5.0            | 194.8        | 20.0           | 6.0         | 32.0         | 22.0          | 36.0                | 3.0                 | 0.23         |
| 412 broccoli casserole                | 306168   | 40      | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0]            | 6       | 9             | 5.0    | 5.0            | 194.8        | 20.0           | 6.0         | 32.0         | 22.0          | 36.0                | 3.0                 | 0.23         |
| 412 broccoli casserole                | 306168   | 40      | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0]            | 6       | 9             | 5.0    | 5.0            | 194.8        | 20.0           | 6.0         | 32.0         | 22.0          | 36.0                | 3.0                 | 0.23         |

### Univariate Analysis
Here, we looked at the distribution of `prop_protein` across all recipes. We wanted to see how much protein were in the recipes on food.com. We observed a right-skewed distribution, where most recipes were low in protein. As the proportion of protein increases, we see less of those recipes.

<iframe
  src="assets/avg_rating_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis
Here, we looked at the relationship between `prop_protein` and `average_rating`, which was our initial question! We observed a distribution that was skewed to the right with many high-rated recipes concentrated at lower protein amounts. Based on our original idea, that the uptick in protein focus in diet, this doesn't align, but when we consider that most recipes on food.com are low in protein already (univariate analysis), this makes more sense. 

<iframe
  src="assets/protein_vs_avg_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates
Here we looked at the relationship between `prop_protein` and `n_ingredients`. We created a pivot table to visualize their connection better. 

| n_ingredients | min  | max   | median | mean  | midpoint |
|---------------|------|-------|--------|-------|----------|
| 1             | 0.0  | 157.0 | 17.0   | 53.09 | 78.5     |
| 2             | 0.0  | 1051.0| 7.0    | 29.29 | 525.5    |
| 3             | 0.0  | 1043.0| 5.0    | 17.46 | 521.5    |
| ...           | ...  | ...   | ...    | ...   | ...      |
| 32            | 38.0 | 94.0  | 94.0   | 80.00 | 66.0     |
| 33            | 8.0  | 8.0   | 8.0    | 8.00  | 8.0      |
| 37            | 59.0 | 59.0  | 59.0   | 59.00 | 59.0     |

We also created a graph, since just looking at the numbers was tricky. 

<iframe
  src="assets/pro_ing.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Here, we see that the mean, median, and min lines are similar the whole way through, for less ingredients and more ingredients. Those values stay very consistent, regardless of the number of ingredients, while only the midpoint and max seem to fluctuate at an overal decreasing trend. This makes sense, as the max will influence the midpoint (it is involved in the midpoint's calculation).

## Assessment of Missingness
### NMAR Analysis
There are three columns in our dataset that contain missing values: `date`, `rating`, and `review`.

We believe that the `review` column is NMAR. Typically, only people who feel strongly about a recipe will come back to write a review after they've cooked it. Either they found it to be extremely good or extremely poor. People who feel neutral about the recipe aren't as likely to write a review. We might also consider that there is a larger barrier to leaving a review rather than a rating, since it requires a written response; many people may feel it's not worth their time. Since the missingness of `review` depends on its values, it is NMAR.

### Missingness Dependency
Here, we are investigating the missingness of the `rating` column. We ran two permutation tests that evaluate its dependency on saturated fat content and the number of steps in the recipes.

**Dependency on Saturated Fat Content**
Similarly to computing the proportion of protein earlier, we found the proportion of saturated fat contributing to calories using the FDA's recommmend amounts (20g per day, 9 calories per gram).

We tested if the missingness of `rating` is dependent on `prop_sat_fat`.

<iframe
  src="assets/sat_fat_by_missing.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of saturated fat for both groups seems roughly the same. 

Null Hypothesis: The missingness of ratings does is not dependent on the saturated fat content.

Alternate Hypothesis: The missingness of ratings is dependent on the saturated fat content.

Test Statistic: The absolute difference of means between the proportion of saturated fat for a group with present ratings and a group with missing ratings.

Significance Level: 0.05

After running our permutation test, we got a p-value of 0.0. It is less than our significance level, so we *reject the null hypothesis.* The missingness of rating is MAR dependent on saturated fat content. 

**Dependency on Number of Tags**
We then tested if the missingness of `rating` is dependent on `n_tags`.

<iframe
  src="assets/n_tags_by_missing.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The average tags for both groups seems roughly the same. 

Null Hypothesis: The missingness of ratings does is not dependent on the number of tags.

Alternate Hypothesis: The missingness of ratings is dependent on the number of tags.

Test Statistic: The absolute difference of means between the average number of tags for a group with present ratings and a group with missing ratings.

Significance Level: 0.05

After running our permutation test, we got a p-value of 1.0. It is greater than our significance level, so we *fail to reject the null hypothesis.* The missingness of rating is not MAR dependent on number of tags. 

## Hypothesis Testing
Here, we go back to our original question: connecting protein to ratings in some way.

Intuitively, we thought that recipes with higher protein might have better ratings. So, we took the `prop_protein` column and transformed it into a new boolean column `is_high_protein`. The FDA defines a food as being high-protein if its proportion of protein is greater than 0.2.

Null Hypothesis: People rate all recipes similarly, regardless of protein content.

Alternative Hypothesis: People rate high-protein recipes higher than low-protein recipes.

Test Statistic: The difference in means between ratings for high-protein vs. low-protein recipes.

Significance Level: 0.05

We ran a permutation test to determine if these two groups' appear similar (are from the same population). We chose a one-sided test, since due to the rise in protein-focused health and fitnessed, we thought that people would likely rate high-protein recipes higher and used the difference in means to account for this.

After running our permutation test, we got a p-value of 1.0. It is greater than our significance level, so we *fail to reject the null hypothesis.* It seems as though that a recipe being high-protein does not have a significant impact (increase) on its rating.

## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis
