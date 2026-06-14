# Machine Learning Analysis: Regression, Dimensionality Reduction & Classification

A three part machine learning study exploring OLS regression stability, PCA/LDA dimensionality reduction on nutritional data, and KNN/QDA classification with feature selection. Each analysis is implemented in a self contained Jupyter Notebook using scikit-learn, NumPy, Pandas, Matplotlib, and Seaborn.

---

## Repository Structure

```
├── P1.ipynb              # Analysis 1 — OLS Regression & Regularization
├── P2.ipynb              # Analysis 2 — PCA & LDA on Nutritional Data
├── P3.ipynb              # Analysis 3 — KNN, QDA & Backward Feature Selection
└── data-analysis.pdf     # Full written analysis with figures
```

---

## Analysis 1 — OLS Stability & Ridge Regularization (`P1.ipynb`)

**Goal:** Evaluate how sensitive Ordinary Least Squares regression is to random data splits, and whether L2 regularization (Ridge) can improve generalization.

### Part 1 — OLS Stability across Randomized Splits
- Ran 500 independent train/test splits (50/50) using random seeds
- Computed the pseudoinverse solution on each training set and recorded the test R²
- **Findings:** R² was approximately normally distributed with a mode of 0.78 and mean of 0.66, but the distribution was left-skewed with an extreme outlier at −0.40. A negative R² indicates the model performed worse than simply predicting the mean, exposing high sensitivity to sampling variation.

### Part 2 — Effect of Noisy Predictors on OLS
- Generated synthetic data with a controlled signal and incrementally added up to 30 noise features
- Averaged training and test R² over 100 trials per noise level
- **Findings:** Training R² climbed from 0.91 (1 noise feature) to 0.97 (30 noise features), while test R² fell from 0.90 to 0.71 — a clear overfitting signature caused by OLS fitting spurious correlations in noisy channels.

### Part 3 — Ridge Regression Stability
- Applied L2 regularization (λ = 0 to 2) using the same 500-split experiment
- **Findings:** Mean test R² rose sharply from 0.65 (λ = 0) to a peak of 0.76 at λ = 0.40, then gradually declined as the penalty became too aggressive. This confirms the bias-variance tradeoff: a moderate penalty reduces variance enough to improve generalization without introducing excessive bias.

**Key Libraries:** `sklearn.linear_model.LinearRegression`, `Ridge`, `numpy`, `matplotlib`, `seaborn`

---

## Analysis 2 — PCA & LDA on Nutritional Data (`P2.ipynb`)

**Goal:** Evaluate unsupervised (PCA) and supervised (LDA) dimensionality reduction on a 46-feature nutritional dataset spanning 14 food categories.

### Part 1 — PCA on the Original (Unstandardized) Dataset
- Features used exclude class labels, text descriptors, and unique identifiers
- **Findings:** Only 5 principal components were needed to capture 95% of variance — a warning sign. The first component was dominated almost entirely by Vitamin A (IU scale), and the second by Lycopene, obscuring all other nutritional relationships. The resulting scatter plot showed apparent but misleading separation between just two food groups, driven entirely by scale artifacts rather than true nutritional patterns.

### Part 2 — PCA with Feature Standardization
- Applied `StandardScaler` before PCA to equalize feature influence
- **Findings:** 28 components were now required to capture 95% of variance, reflecting a much more balanced spread of information. Feature contributions became evenly distributed, and the 2-component scatter revealed meaningful nutritional axes: PC1 corresponded to overall nutrient density (vitamins positive, water negative) and PC2 to fat content (lipids/fatty acids positive, water/vitamins negative). Food groups — beverages, general foods, baby foods — separated along these nutritional axes.

### Part 3 — LDA vs. PCA Comparison
- Applied LDA to the standardized dataset using class labels to maximize between-class separation
- **Findings:** LDA required only 8 discriminant components to reach 95% explained variance, versus 28 for PCA, indicating high linear separability in the feature space. The 2-component LDA projection produced far tighter, more distinct clusters than PCA. LD1 was driven by water and protein content; LD2 by vitamin A and retinol. Cluster structure on the held-out validation set mirrored the training projection, confirming the model did not overfit on noise.

**Key Libraries:** `sklearn.decomposition.PCA`, `sklearn.discriminant_analysis.LinearDiscriminantAnalysis`, `sklearn.preprocessing.StandardScaler`, `sklearn.impute.SimpleImputer`, `scipy.sparse`, `matplotlib`, `seaborn`

---

## Analysis 3 — KNN, QDA & Backward Feature Selection (`P3.ipynb`)

**Goal:** Compare a non-parametric classifier (KNN) and a parametric classifier (QDA) on the nutritional dataset, then use Backward Sequential Feature Selection to identify the most informative features.

### Part 1 — K-Nearest Neighbors Classification
- Standardized features before training; evaluated across k = 1 to 20 neighbors
- **At k = 1:** Training accuracy was 100% (each point is its own neighbor) and validation accuracy reached ~87%. Misclassifications occurred only between nutritionally similar groups (lamb/beef; fruits/vegetables), suggesting the model learned meaningful boundaries.
- **As k increased:** Both training and validation accuracy steadily declined, converging near 80% by k = 17, indicating that averaging too many distant neighbors introduces underfitting.

### Part 2 — Quadratic Discriminant Analysis Classification
- Applied QDA with regularization parameter λ swept from 0 to 1 to stabilize covariance estimates
- **Findings:** Initial training accuracy (~75%) and validation accuracy (~67%) were substantially lower than KNN, suggesting that food groups do not follow the multivariate Gaussian distributions assumed by QDA. A local validation accuracy peak emerged near λ = 0.90, where regularization briefly stabilized covariance matrices and optimized the bias-variance tradeoff before excessive rigidity collapsed predictive power.

### Part 3 — Backward Sequential Feature Selection
- Selected the best model (KNN, k = 1) as the wrapper estimator
- Used 5-fold cross-validation with iterative feature pruning to identify the 10 most informative features
- Excluded `Unnamed: 0` (row index) to prevent data leakage — including it inflated accuracy to ~95% by letting the model exploit the original sorted ordering of the data rather than nutritional content
- **Selected features:** Water, Protein, Ash, Potassium, Selenium, Riboflavin, Food Folate, FA_Sat, FA_Poly, GmWt_1
- **Interpretation:** These features align with known nutritional separators. Water content distinguishes high-moisture groups (fruits, vegetables) from dry products (baked goods). Protein separates meat groups from cereals and sweets. Fatty acid profiles (FA_Sat, FA_Poly) differentiate animal products from plant-based categories.

**Key Libraries:** `sklearn.neighbors.KNeighborsClassifier`, `sklearn.discriminant_analysis.QuadraticDiscriminantAnalysis`, `sklearn.feature_selection.SequentialFeatureSelector`, `sklearn.preprocessing.StandardScaler`, `matplotlib`, `seaborn`

---

## Setup & Requirements

```bash
pip install numpy pandas scikit-learn matplotlib seaborn scipy
```

Python 3.8+ recommended. Run each notebook independently — they have no cross-dependencies.

---

## Key Takeaways

| Topic | Finding |
|---|---|
| OLS sensitivity | High variance across splits; single bad split yields R² < 0 |
| Noisy features | Each additional noise predictor widens the train/test R² gap |
| Ridge regularization | Optimal at λ ≈ 0.40; improves mean test R² from 0.65 → 0.76 |
| PCA without scaling | Dominated by high-range features (Vitamin A, Lycopene); misleading |
| PCA with scaling | 28 components for 95% variance; nutritionally interpretable axes |
| LDA vs. PCA | LDA achieves 95% in 8 components; far tighter clusters |
| KNN vs. QDA | KNN (87% valid.) outperforms QDA (67% valid.); data is non-Gaussian |
| Feature selection | 10 features sufficient; water, protein, and fatty acids are key separators |

---

## References

- Buitinck et al. (2013). *API design for machine learning software: experiences from the scikit-learn project.* ECML PKDD Workshop. https://arxiv.org/abs/1309.0238
- Galarnyk, M. (2024). *PCA in Python tutorial with scikit-learn.* Built In. https://builtin.com/machine-learning/pca-in-python
- Singh, R. (2021). *Forward and backward subset selection method.* Kaggle. https://www.kaggle.com/code/jurk06/forward-and-backward-subset-selection-method
- Hunter, J. (2003). *Matplotlib API Reference.* https://matplotlib.org/stable/api/index.html
- Waskom, M. (2013). *Seaborn API Reference.* https://seaborn.pydata.org/api.html

> Documentation written with AI assistance. All analysis, code, and results are the author's own work.
