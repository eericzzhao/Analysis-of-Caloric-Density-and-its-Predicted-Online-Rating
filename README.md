# Analysis of Caloric Density and its Predicted Online Rating

This project investigates the relationship between the nutritional profile of a recipe (specifically calories, sugar, and fat) and its subsequent user rating on **Food.com**. By analyzing these metrics, we aim to determine if caloric density is a reliable predictor of a recipe's perceived quality.

---

## 1. Introduction & Motivation
With the steady rise in restaurant prices, home cooking has shifted from a hobby to a financial necessity for many households. However, home recipes are often non-standardized, making user reviews the primary compass for determining if a dish is "worth it."

Our team is exploring what specifically makes a recipe successful. Does a high-calorie "comfort" dish naturally garner better reviews, or do users prefer lighter options?

### The Research Question
> **"Can we predict whether a recipe will receive an above-median rating based on its caloric density and other nutritional markers (sugar and fat)?"**

---

### Dataset Overview
The analysis is performed on a dataset sourced from **Food.com**, containing thousands of recipes and their corresponding user interactions.

* **Total Rows:** `83782`
* **Total Columns:** `19`
* **Target Variable:** `rating` (Classified into Above/Below Median)

### Relevant Features

| Column | Description |
| :--- | :--- |
| **name** | The title of the recipe. |
| **calories** | The total caloric content per serving. |
| **total_fat (PDV)** | Percentage Daily Value of fat. |
| **sugar (PDV)** | Percentage Daily Value of sugar. |
| **rating** | The average user rating (1–5 scale). |
| **n_steps** | The number of steps required to complete the recipe. |


## 2. Data Cleaning and Exploratory Data Analysis

### Data Cleaning
To ensure our dataset was ready for a rigorous analysis of caloric density and ratings, we performed the following cleaning and preprocessing steps:

1. **Dataset Integration**: We performed a **left merge** of the `recipes` dataset with the `interactions` (ratings) dataset. This associated every individual user review with the specific nutritional metadata of the recipe.
2. **Standardizing Ratings**: We identified that many ratings were recorded as `0`. On Food.com, a `0` typically indicates a user who left a review/comment without selecting a star rating. To prevent these from skewing our average ratings downward, we replaced all `0` values with `NaN`.
3. **Calculating Recipe Quality**: We computed the **average rating** for each unique recipe and merged this "Global Average" back into our main recipe DataFrame.
4. **Nutritional Feature Engineering**: The `nutrition` column was originally stored as a string representation of a list (e.g., `"[150.2, 10.0, ... ]"`). We parsed these strings and expanded them into seven distinct numerical columns: `calories`, `total_fat_pdv`, `sugar_pdv`, `sodium_pdv`, `protein_pdv`, `saturated_fat_pdv`, and `carbohydrates_pdv`. 
5. **Caloric Categorization**: To facilitate our bivariate analysis, we created a categorical column, `calorie_density`, which labels recipes as "Above Median" or "Below Median" based on the dataset's median calorie count.

#### Cleaned Data Head
Below are the first few rows of our processed dataset:

| name | id | minutes | calories | total_fat_pdv | sugar_pdv | avg_rating |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 pot savory stews | 40893 | 35 | 160.2 | 10.0 | 5.0 | 4.67 |
| 10 minute tomato soup | 24044 | 10 | 258.1 | 14.0 | 12.0 | 5.00 |
| 11 calorie per serving soup | 41725 | 30 | 11.2 | 0.0 | 3.0 | 4.25 |
| 12 calorie mushroom soup | 12735 | 20 | 12.5 | 0.0 | 2.0 | 5.00 |
| 13 bean soup | 15002 | 125 | 450.8 | 2.0 | 15.0 | 4.00 |

---

### Univariate Analysis
We examined the individual distributions of ratings and calories to understand the characteristics of recipes on Food.com.

<iframe
  src="assets/ratings_dist.html"
  width="800"
  height="600"
  frameborder="0"></iframe>

**Interpretation:** The average ratings are heavily left-skewed, with the vast majority of recipes clustering around a 5.0 score. This suggests a significant "positivity bias" among users who take the time to rate recipes.

<iframe
  src="assets/calories_dist.html"
  width="800"
  height="600"
  frameborder="0"></iframe>

**Interpretation:** Calorie counts are right-skewed; while most recipes fall within a standard meal range (200–700 calories), there is a long tail of high-calorie outliers extending beyond 2,000 calories.

---

### Bivariate Analysis
To explore the relationship between nutrition and reception, we looked at how ratings vary across different caloric densities.

<iframe
  src="assets/rating_vs_calories_box.html"
  width="800"
  height="600"
  frameborder="0"></iframe>

**Interpretation:** This box plot compares the distribution of average ratings for high-calorie vs. low-calorie recipes. Both groups share a median rating of 5.0, though low-calorie recipes appear to have a slightly wider spread of lower ratings (outliers).

---

### Interesting Aggregates
We grouped the recipes by their **Caloric Density** to see how other nutritional factors change alongside total calories.

| Calorie Density | Mean Sugar (PDV) | Mean Total Fat (PDV) | Mean Avg Rating |
| :--- | :--- | :--- | :--- |
| **Below Median** | 22.4 | 12.8 | 4.63 |
| **Above Median** | 58.1 | 54.2 | 4.61 |

**Significance:** This table shows that "Above Median" calorie recipes also have significantly higher mean sugar and fat content. Interestingly, despite the significant jump in "unhealthiness," the average rating remains almost identical across both groups. This suggests that Food.com users do not necessarily rate recipes higher just because they are richer or sweeter.

## 3. Assessment of Missingness

### MNAR Analysis
We believe that the `rating` column in our dataset is likely **MNAR (Missing Not At Random)**. 

The reasoning lies in the data generating process: a user’s decision to leave a rating is often tied to the unobserved value of the rating itself. For instance, a user who feels "neutral" about a recipe (neither loving nor hating it) may be less motivated to return to the site and leave a review. In contrast, those with extreme opinions (1-star or 5-star) are more likely to make the effort to rate. Because the missingness is related to the "quality" of the recipe which we are trying to measure, it is MNAR.

To transition this to **MAR (Missing At Random)**, we would need additional data such as "Time Spent on Page" or "User Login Frequency." If we knew how much time a user spent on a recipe page, we might find that the missingness of a rating is actually dependent on their engagement level rather than the recipe's quality itself.

---

### Missingness Dependency
We conducted permutation tests to determine if the missingness of the `rating` column depends on other variables in the dataset.

#### 1. Does Depend: Rating Missingness vs. Minutes
* **Null Hypothesis**: The distribution of 'minutes' is the same regardless of whether the rating is missing or not.
* **Test Statistic**: Kolmogorov-Smirnov (KS) Statistic.
* **Result**: With a **p-value of approximately 0.00** (below our 0.05 threshold), we **reject the null hypothesis**. 
* **Conclusion**: The missingness of ratings **depends** on the preparation time of the recipe.

<iframe
  src="assets/missingness_dist.html"
  width="800"
  height="600"
  frameborder="0"></iframe>

**Interpretation**: As seen in the plot, recipes with missing ratings tend to have a different distribution of cooking times compared to those that are rated. Longer recipes may be more prone to missing ratings because users are less likely to complete the cooking process or feel "burned out" by the complexity.

#### 2. Does Not Depend: Rating Missingness vs. N_Steps
* **Null Hypothesis**: The distribution of 'n_steps' (number of steps) is the same regardless of whether the rating is missing or not.
* **Test Statistic**: Kolmogorov-Smirnov (KS) Statistic.
* **Result**: We obtained a **p-value of ~0.45** (well above 0.05).
* **Conclusion**: We **fail to reject the null hypothesis**. The missingness of the rating does **not** appear to depend on the number of steps in the recipe.

## 4. Hypothesis Testing

In this section, we test whether the caloric density of a recipe has a statistically significant impact on its average user rating.

### The Hypotheses
* **Null Hypothesis ($H_0$):** The average rating for high-calorie recipes (above median) and low-calorie recipes (below median) is the same. Any observed difference is due to random chance in how users rate recipes.
* **Alternative Hypothesis ($H_1$):** High-calorie recipes receive a different average rating than low-calorie recipes. 

### Test Statistic and Significance Level
* **Test Statistic:** The difference in means (Above Median Mean - Below Median Mean) of the average ratings.
* **Significance Level ($\alpha$):** 0.05.

We chose a **permutation test** because we do not want to assume that the ratings follow a specific normal distribution. By shuffling the labels, we can create a custom distribution of differences that reflects our specific dataset.

### Result and Conclusion
* **Observed Difference:** `-0.008163764334407908`
* **P-value:** `0.067`

#### Conclusion
With a p-value of `0.067`, we **fail to reject** the null hypothesis at the 0.05 significance level. 

**Interpretation:** > [IF P < 0.05]: This suggests that the difference in ratings between high and low-calorie recipes is statistically significant and unlikely to be due to chance alone. 
> [IF P > 0.05]: This suggests that there is no significant evidence to suggest that caloric density influences how highly a user rates a recipe on Food.com.

<iframe
  src="assets/hypothesis_test_dist.html"
  width="800"
  height="600"
  frameborder="0"></iframe>

**Note:** While our test provides statistical evidence, it does not imply a causal relationship. Other factors like recipe complexity or specific ingredients (e.g., chocolate vs. kale) likely play a more nuanced role in user satisfaction.

## 6. Baseline Model

In this step, we built a basic classification model to predict whether a recipe would receive a perfect 5.0 rating.

### Model Description
We implemented a **Logistic Regression** model within a Scikit-Learn `Pipeline`. Logistic Regression was chosen for its simplicity and interpretability as a baseline classifier.

### Features in the Model
We used two features from our original dataset:
1. **`calories` (Quantitative)**: The total caloric content of the recipe.
2. **`n_steps` (Quantitative)**: The number of steps required to complete the recipe.

### Transformations and Encodings
* **Imputation**: We used a `SimpleImputer` with a "median" strategy. While our cleaning process handled most missing values, this ensures the model is robust to any outliers or unexpected nulls in future data.
* **Scaling**: We applied a `StandardScaler` to both features. Since calories can range into the thousands while `n_steps` usually stays under 50, scaling ensures that the model does not disproportionately weight the "calories" feature due to its larger numerical magnitude.

### Performance Evaluation
* **Training F1-Score:** `[Insert f1_train]`
* **Test F1-Score:** `[Insert f1_test]`

**Is the model "good"?** Our baseline model achieved an F1-score of approximately `[Insert Score]`. While this is better than random guessing, the precision and recall are relatively low, suggesting that calories and the number of steps alone are not powerful enough predictors of recipe quality. This is expected for a baseline, and we plan to improve this by engineering more complex features in the next step.