# Investigation of Nutrition on Ratings and Calories
By: Karina Zambani and Ivan Chen

## Introduction
In today's diet culture, we are seeing the rapid rise of focus placed on protein. You can't watch a single recipe on TikTok or a gym Youtube video without protein being emphasized for health and fitness. Because of this, we were interested in if **the protein content of a recipe would affect its rating.**

As such, we are exploring two datasets to answer this question. Both are sourced from [food.com](food.com), which is a popular website for recipes where users can leave reviews.

The first dataset is `recipes`, which includes 83782 unique recipes with the following 10 columns:
| Column         | Description                                                                                                                   |
|----------------|-------------------------------------------------------------------------------------------------------------------------------|
| `name`         | Recipe name                                                                                                                  |
| `id`           | Recipe ID                                                                                                                    |
| `minutes`      | Minutes to prepare recipe                                                                                                    |
| `contributor_id` | User ID who submitted this recipe                                                                                           |
| `submitted`    | Date recipe was submitted                                                                                                    |
| `tags`         | Food.com tags for recipe                                                                                                     |
| `nutrition`    | Nutrition information in the form `[calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]`; PDV stands for “percentage of daily value” |
| `n_steps`      | Number of steps in recipe                                                                                                    |
| `steps`        | Text for recipe steps, in order                                                                                              |
| `description`  | User-provided description                                                                                                    |
| `ingredients`  | Text for recipe ingredients                                                                                                  |
| `n_ingredients`| Number of ingredients in recipe                                                                                              |

The second dataset is `interactions`, which includes 731927 user reviews with the following 5 columns:
| Column      | Description                 |
|-------------|-----------------------------|
| `user_id`   | User ID                     |
| `recipe_id` | Recipe ID                   |
| `date`      | Date of interaction         |
| `rating`    | Rating given                |
| `review`    | Review text                 |

This analysis will allow us to determine the weight that people might give protein to when it comes to cooking, and if explored further, give some idea to what kinds of people are more health-conscious.

## Data Cleaning and Exploratory Data Analysis
### Data Cleaning Steps

1. Performed a left merge of the `recipes` and `interactions` datasets on their unique recipe IDs. Each recipe is now matched to its review.
2. Filled any ratings valued "0" with np.nan. Ratings are only offered on a range of 1-5, so a rating of 0 is an indication that the value is actually missing. When performing aggregations later on, we want to avoid any missing values dragging values in a certain direction, so we replaced them with null values.
3. Computed and added a column for `average_rating` of each recipe. This is to compress down the ratings for each recipe into a single value we can work with.
4. There are 3 columns that appear to contain lists of values, but are actually strings. For the `tags`, `nutrition`, and `steps` columns, we converted these values to actual lists of strings and floats. We want to be able to work with individual nutrition values and assess specific tags. 
5. Separated the `nutrition` column list of values into separate columns. This lets us assess specific types of nutrition in our analysis. 
6. Computed and added a column for `prop_protein` which contains the proportion of protein of a recipe that contributes to its total calories. The FDA (Food and Drug Association) state that the recommended amount of protein per day is 50g and that each gram contains 4 calories. We used these numbers to calculate this proportion from the `protein (PDV)` column. Our analysis aims to connect protein to ratings, so we needed a way to evaluate protein content in a standardized way. 

Our cleaned dataset is displayed below. It only shows the first 5 rows, but there are 83782 total. We have also ommitted columns not relevant to our analysis, but all of the columns are: ['name', 'id', 'minutes', 'contributor_id', 'submitted', 'tags',
       'nutrition', 'n_steps', 'steps', 'description', 'ingredients',
       'n_ingredients', 'user_id', 'recipe_id', 'date', 'rating', 'review',
       'average_rating', 'calories (#)', 'total fat (PDV)', 'sugar (PDV)',
       'sodium (PDV)', 'protein (PDV)', 'saturated fat (PDV)',
       'carbohydrates (PDV)', 'prop_protein', 'rating_missing', 'prop_sat_fat',
       'n_tags', 'is_high_protein']

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

## Assessment of Missingness

## Hypothesis Testing

## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis
