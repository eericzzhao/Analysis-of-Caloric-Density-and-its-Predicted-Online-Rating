# Analysis of Caloric Density and its Predicted Online Rating

This project investigates the relationship between the nutritional profile of a recipe (specifically calories, sugar, and fat) and its subsequent user rating on **Food.com**. By analyzing these metrics, we aim to determine if caloric density is a reliable predictor of a recipe's perceived quality.

---

## 1. Introduction & Motivation
With the steady rise in restaurant prices, home cooking has shifted from a hobby to a financial necessity for many households. However, home recipes are often non-standardized, making user reviews the primary compass for determining if a dish is "worth it."

Our team is exploring what specifically makes a recipe successful. Does a high-calorie "comfort" dish naturally garner better reviews, or do users prefer lighter options?

### The Research Question
> **"Can we predict whether a recipe will receive an above-median rating based on its caloric density and other nutritional markers (sugar and fat)?"**

---

## 2. Dataset Overview
The analysis is performed on a dataset sourced from **Food.com**, containing thousands of recipes and their corresponding user interactions.

* **Total Rows:** `[Enter Number]`
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

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning
To transform the raw Food.com data into a format suitable for analysis, we performed the following cleaning steps:

1. **Merging & Aggregation**: We joined the `recipes` dataset with the `interactions` dataset. Because our analysis focuses on recipe-level quality, we calculated the **average rating** for each recipe and merged this back into the main recipes table.
2. **Handling Placeholder Ratings**: We identified that many ratings were recorded as `0`. On Food.com, users can leave a text review without selecting a star rating, which results in a `0` in the data. To avoid skewing our averages toward zero, we replaced these values with `NaN`.
3. **Nutritional Feature Extraction**: The `nutrition` column was originally a string of lists. We parsed this column to create individual numeric columns for **Calories, Total Fat, Sugar, Sodium, Protein, Saturated Fat, and Carbohydrates**. This allows us to perform mathematical operations on specific nutrients.
4. **Unit Verification**: All nutritional values (except calories) were confirmed to be in **Percentage Daily Value (PDV)**, ensuring a standardized scale for comparing "sweetness" or "fatness" across different serving sizes.

#### Cleaned Data Header (First 5 Rows)
| name | id | minutes | calories | total_fat | sugar | avg_rating |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 pot savory stews | 40893 | 35 | 160.2 | 10.0 | 5.0 | 4.67 |
| 10 minute tomato soup | 24044 | 10 | 258.1 | 14.0 | 12.0 | 5.00 |
| ... | ... | ... | ... | ... | ... | ... |

---

### Univariate Analysis

In this section, we examine the distributions of our target variable (ratings) and our primary predictor (calories).

#### Distribution of Average Ratings
<iframe
  src="assets/ratings_dist.html"
  width="800"
  height="600"
  frameborder="0"></iframe>

**Interpretation:** The distribution of average ratings is significantly left-skewed, with a massive concentration of recipes receiving a perfect 5.0 score. This suggests a "positivity bias" common in online recipe platforms, where users are more likely to rate recipes they enjoyed.

#### Distribution of Calories
<iframe
  src="assets/calories_dist.html"
  width="800"
  height="600"
  frameborder="0"></iframe>

**Interpretation:** Calories follow a right-skewed distribution. While the majority of recipes fall between 200 and 600 calories (typical for a standard meal), there is a "long tail" of high-calorie recipes that likely represent large-batch items or high-density desserts.

---

## Data Cleaning and Exploratory Data Analysis

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