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

### Bivariate Analysis
We explored the relationship between a recipe's "unhealthiness" (sugar content) and its user reception.

<iframe
  src="assets/sugar_vs_rating.html"
  width="800"
  height="600"
  frameborder="0"></iframe>

**Interpretation**: While one might expect "sweeter" recipes to have higher ratings, the scatter plot shows a fairly uniform distribution of high ratings across all sugar levels. This suggests that sweetness alone isn't a guaranteed driver of high ratings.