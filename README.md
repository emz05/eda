# IMDb Movie Analysis

Exploratory data analysis and hypothesis testing on IMDb movie and ratings data, examining how demographics, budget, genre, and release timing relate to movie ratings and revenue.

---

## Project Structure

```
.
├── P1.ipynb              # EDA — genre/gender/age preferences, budget-rating relationship, seasonal patterns
├── P2.ipynb              # Demographic epicycle — age and gender effects on action movie ratings
├── P3.ipynb              # Budget epicycle — budget influence on gross revenue by genre (ANOVA)
├── IMDb movies.csv       # Movie metadata (title, year, genre, budget, gross income, etc.)
├── IMDb ratings.csv      # Demographic rating breakdowns (age group × gender average votes)
└── README.md
```

---

## Data

Two CSV files are required in the project root:

| File | Description |
|---|---|
| `IMDb movies.csv` | Movie metadata: title, year, date published, genre, budget, USA/worldwide gross income |
| `IMDb ratings.csv` | Ratings broken down by gender (male/female) and age group (<18, 18–29, 30–45, 45+) |

---

## Dependencies

Install all required packages:

```bash
pip install pandas numpy matplotlib seaborn scipy statsmodels
```

Or install per notebook:

```bash
# P1 and P2
pip install scipy

# P3 (adds statsmodels for two-way ANOVA)
pip install scipy statsmodels
```

---

## Notebooks

### P1 — EDA (`P1.ipynb`)

Answers five exploratory questions across three topic areas:

**Genre & Gender**
Computes weighted average ratings (minimum 100 votes per gender) for the top 6 genres and compares male vs. female averages via a dumbbell plot. Females rate consistently higher across all genres; the largest gap is Romance (+0.25) and the smallest is Crime (+0.14). The overall magnitude is small relative to the 1–10 scale, suggesting rating style differences rather than strong genre-specific preferences.

**Age & Genre**
Constructs a genre × age-group heatmap of average ratings. Ratings decline monotonically with age across all genres (Drama range: 7.11 for <18 vs. 6.42 for 45+). Genre preference ordering is stable across age groups: Drama > Romance ≈ Crime > Comedy > Action > Thriller.

**Budget & Ratings**
Bins movies into seven budget tiers and visualizes the rating distribution using violin plots. Higher budget tiers show tighter, higher-median distributions. Variance is largest in the lowest budget tier, confirming that budget increases the floor of rating quality but is not the sole determinant of success.

**Genre Trends Over Time (1900–2020)**
Computes each genre's yearly share of total releases (smoothed with a 10-year rolling window) and fits a linear regression per genre. Action (r = 0.85) and Thriller (r = 0.98) show the strongest positive trends. Drama (r = −0.60) and Romance (r = −0.57) decline in share despite Drama remaining the most-released genre overall.

**Seasonal Release Patterns**
Maps release month to season and computes genre share within each season. Thriller, Crime, and Action are most concentrated in summer (17%, 14%, 14%). Romance and Comedy peak in winter (17%, 37%). This suggests release timing interacts with genre.

---

### P2 — Demographic Epicycle (`P2.ipynb`)

Focuses on action-type movies (genres: Action, Adventure, Thriller, War).

**Part 1 — Interaction Check**
Computes weighted average ratings for four groups (young male, old male, young female, old female) where young = ages 0–29 and old = 30+. A line plot shows parallel slopes for both genders from young to old, and a scatter plot of male age-effect vs. female age-effect shows clustering near the identity line. A paired t-test on age-difference scores (young − old) per movie fails to reject the null of no gender × age interaction (p = 0.21). Age and gender act as independent additive factors.

**Part 2 — Independent Effects**

*Age effect:* Paired t-test on young vs. old ratings per movie: t = 20.67, p = 6.29 × 10⁻⁷¹. Younger viewers rate action movies 0.16 points higher on average (1.6% of scale). Null rejected.

*Gender effect:* Paired t-test on female vs. male ratings per movie: t = 13.82, p = 1.45 × 10⁻³⁷. Female viewers rate action movies 0.20 points higher on average (2.0% of scale). Null rejected.

Both effects are statistically significant but small in absolute terms. Rating-style differences (leniency bias) cannot be ruled out as a partial explanation.

---

### P3 — Budget Epicycle (`P3.ipynb`)

Examines whether budget affects worldwide gross revenue, and whether this relationship differs between Drama and Action.

**Part 1 — Mean Revenue by Budget and Genre**
Budget is binarized at each genre's median into low/high. High-budget Action films earn ~$158M more on average than low-budget Action; high-budget Drama earns ~$48M more. The raw mean difference between genres is ~$109M in Action's favor.

**Part 2 — Distribution Diagnostics**
Boxplots on both raw and log-transformed revenue reveal severe right skew in the raw data. Mean-to-median ratios range from 2.01 (Action High) to 12.10 (Drama Low), confirming that outliers distort the Part 1 means. Log transformation is required before formal testing.

**Part 3 — Two-Way ANOVA on Log Revenue**
A Type II two-way ANOVA on log(worldwide gross income) with factors Genre and Budget:

| Term | F | p |
|---|---|---|
| Genre | 635.38 | 1.83 × 10⁻¹³⁵ |
| Budget | 3512.12 | < 0.001 |
| Genre × Budget | 0.0002 | 0.989 |

Both main effects are highly significant. The interaction term is not (p = 0.989), meaning budget provides a proportionally similar revenue boost across both genres on the log scale. The large raw-dollar difference seen in Part 1 is an artifact of outliers and baseline revenue differences between genres, not a differential budget sensitivity.

---

## Key Findings Summary

| Analysis | Finding |
|---|---|
| Gender × Genre | Females rate ~0.14–0.25 pts higher than males across genres; effects are small and consistent (likely rating-style differences) |
| Age × Genre | Younger viewers rate higher across all genres; decline is monotonic with age |
| Budget × Rating | Higher budgets associate with higher and less variable ratings, but low-budget films can still achieve high ratings |
| Genre Trends | Action and Thriller growing in share; Drama and Romance declining since ~1960 |
| Seasonal Patterns | Action genres peak in summer; Romance/Comedy peak in winter |
| Budget × Revenue | Budget significantly boosts revenue regardless of genre (no interaction); genre sets the revenue baseline |
| Age/Gender on Action | Both age (Δ = 0.16) and gender (Δ = 0.20) independently affect ratings; no interaction detected |

---

## Reproducing the Analysis

1. Clone the repo and place `IMDb movies.csv` and `IMDb ratings.csv` in the project root.
2. Install dependencies (see above).
3. Run the notebooks in order: `P1.ipynb` → `P2.ipynb` → `P3.ipynb`. Each notebook is self-contained and re-loads the data independently.

---

## Limitations

- **Causal claims are not supported.** All findings are observational. Correlations between budget and revenue may reflect studio selection effects (studios greenlight larger budgets for projects they already expect to succeed).
- **Rating-style confounding.** Observed gender and age differences in average ratings may reflect systematic leniency or severity biases, not genuine preference differences.
- **Currency normalization absent.** Budget and gross income values span decades and are not adjusted for inflation, which affects budget-tier categorization and revenue comparisons.
- **Genre is multi-label.** Movies belong to multiple genres; analysis treats genre membership as non-exclusive (binary indicator columns), so totals across genres exceed 100%.
- **Seasonal analysis conflates release count with performance.** The heatmap shows percentage of movies released per season, not average rating by season. High release counts do not imply high average quality.

---

## References

- Hunter, J. (2003). Matplotlib API Reference. https://matplotlib.org/stable/api/index.html
- McKinney, W. (2009). Pandas API Reference. https://pandas.pydata.org/docs/reference/index.html
- Waskom, M. (2013). Seaborn API Reference. https://seaborn.pydata.org/api.html
- GeeksforGeeks (2025). How to perform a two-way ANOVA in Python. https://www.geeksforgeeks.org/machine-learning/how-to-perform-a-two-way-anova-in-python/

> Documentation written with AI assistance. All analysis, code, and results are the author's own work.
