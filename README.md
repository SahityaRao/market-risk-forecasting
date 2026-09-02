# Enhanced Hybrid Machine Learning Ensembles for Short-Horizon Market Risk Forecasting

## Project Overview

Financial markets can experience sudden short-term declines that are difficult to anticipate using traditional statistical approaches. This project develops a hybrid machine learning framework for identifying elevated downside risk in the S&P 500 over a short forecasting horizon.

The framework combines market-price information, volatility measures, cross-asset relationships, distributional statistics, and time-series characteristics to predict whether the S&P 500 will experience a significant decline over the following five trading days.

Three complementary machine learning models — Multi-Layer Perceptron (MLP), XGBoost, and CatBoost — are combined into a probability-based ensemble. The resulting risk probabilities are further analyzed using SHAP explainability and translated into risk-based trading signals.

The project evaluates both the predictive performance of the ensemble and the effectiveness of its risk signals through a portfolio trading strategy.

---

## Objective

The primary objective is to develop a short-horizon market-risk forecasting system that:

- Predicts significant S&P 500 downside events over a 5-day horizon.
- Captures nonlinear relationships across multiple financial indicators.
- Combines heterogeneous machine learning models into an ensemble.
- Identifies the features driving risk predictions using SHAP.
- Stratifies observations into different levels of predicted market risk.
- Evaluates the resulting signals through a portfolio trading strategy.

---

## Problem Definition

Let \(P_t\) denote the S&P 500 closing price at time \(t\). The daily log return is defined as:

$$
r_t = \ln\left(\frac{P_t}{P_{t-1}}\right)
$$

The target variable represents whether the S&P 500 experiences a cumulative decline of at least 1% over the following five trading days:

$$
Y_t =
\begin{cases}
1, & \text{if } \sum_{i=1}^{5} r_{t+i} \leq -0.01 \\
0, & \text{if } \sum_{i=1}^{5} r_{t+i} > -0.01
\end{cases}
$$

Thus:

- \(Y_t = 1\): a significant 5-day downside event occurs after time \(t\).
- \(Y_t = 0\): no significant 5-day downside event occurs.
- The model uses information available at time \(t\) to predict the outcome over \(t+1,\ldots,t+5\).

This formulation makes the task a **5-day-ahead binary market-risk classification problem**.

---

## Tech Stack

- **Language:** Python
- **Data Analysis:** Pandas, NumPy
- **Visualization:** Matplotlib, Seaborn
- **Statistical Analysis:** SciPy, Statsmodels
- **Machine Learning:** Scikit-learn, XGBoost, CatBoost
- **Model Explainability:** SHAP
- **Data Input:** Excel / OpenPyXL
- **Environment:** Jupyter Notebook

---

## Methodology

The overall workflow consists of:

1. Exploratory Data Analysis
2. Financial feature engineering
3. Feature preprocessing and selection
4. Hybrid machine learning ensemble
5. Model interpretability using SHAP
6. Risk stratification
7. Trading strategy construction
8. Portfolio performance evaluation

---

## Exploratory Data Analysis

The EDA stage examines:

- Historical S&P 500 price trends
- Daily return distributions
- Correlation between financial instruments
- Class imbalance
- Rolling volatility and mean returns
- Return autocorrelation
- Crash-event frequency
- Differences in market characteristics between risk and non-risk observations

---

## Feature Engineering

The model incorporates multiple categories of financial features to capture different dimensions of market behavior.

### Rolling Statistical Features

Rolling-window statistics are calculated over multiple horizons, including:

- Rolling volatility
- Rolling skewness
- Rolling kurtosis
- Rolling entropy

These features capture changes in market variability and the shape of return distributions.

### Time-Series Features

The framework incorporates:

- Hurst exponent
- Time-series dependence characteristics
- Distributional characteristics

The Hurst exponent is calculated over short-, medium-, and long-term windows to capture changes in market persistence and dependence.

### Cross-Asset Features

Rolling relationships between individual assets and the S&P 500 are incorporated through:

- Rolling correlation
- Rolling beta

These features capture changes in cross-asset dependence and market sensitivity.

### Distributional Features

KL divergence features compare the recent 21-day return distribution with a longer 126-day historical reference distribution.

This helps identify changes in the underlying return-generating behavior.

---

## Feature Selection

The feature-processing pipeline consists of:

1. Forward filling and removal of invalid observations.
2. Standardization using `StandardScaler`.
3. Removal of near-zero variance features.
4. Removal of highly correlated features.
5. Mutual information-based feature selection.

The final feature set is used as input to the ensemble models.

---

## Hybrid Ensemble Architecture

The framework combines three machine learning classifiers.

### 1. Multi-Layer Perceptron (MLP)

A neural network with two hidden layers:

- 64 neurons
- 32 neurons
- ReLU activation
- Adam optimizer

### 2. XGBoost

A gradient-boosted decision-tree model configured with:

- 200 estimators
- Maximum depth of 5
- Learning rate of 0.05
- 80% subsampling

### 3. CatBoost

A gradient-boosting model configured with:

- 200 iterations
- Depth of 5
- Learning rate of 0.05

### Ensemble Prediction

Each model produces a probability of a downside event.

The final ensemble probability is calculated as:

<p align="center">
  <img src="https://latex.codecogs.com/svg.image?\color{white}P_{\mathrm{ensemble}}=\frac{P_{\mathrm{MLP}}+P_{\mathrm{XGBoost}}+P_{\mathrm{CatBoost}}}{3}" alt="Ensemble probability equation">
</p>

A predicted probability of at least 0.5 is classified as a downside-risk event.

---

## Model Interpretability

SHAP (SHapley Additive exPlanations) is used to interpret the XGBoost component of the ensemble.

The analysis examines:

- Overall feature importance
- Mean SHAP values for crash and non-crash observations
- The most influential features
- Feature contributions for the latest observation

This provides insight into which market characteristics contribute to elevated predicted downside risk rather than treating the model as a black box.

---

## Risk Stratification

Predicted crash probabilities are divided into five risk quintiles.

The quintiles are used to examine the relationship between predicted risk and subsequent 5-day S&P 500 returns.

The analysis compares:

- Lower-risk observations
- Higher-risk observations
- Average subsequent 5-day returns across risk groups

This provides an additional evaluation of whether higher predicted risk corresponds to weaker subsequent market performance.

---

## Trading Strategy

The predicted crash probability is converted into a continuous trading position:

$$
Position_t = 1 - 2P_{\text{crash},t}
$$

This produces:

- Positive positions when predicted crash probability is low.
- Negative positions when predicted crash probability is high.
- Positions close to zero when predicted risk is approximately 50%.

The resulting position is applied to daily S&P 500 returns to evaluate the performance of the model-driven strategy.

---

## Results

### Ensemble Classification Performance

| Metric | Score |
|---|---:|
| Accuracy | 95.22% |
| Precision | 98.56% |
| Recall | 79.84% |
| F1 Score | 88.21% |
| ROC AUC | 99.47% |

### Confusion Matrix

| | Predicted Non-Crash | Predicted Crash |
|---|---:|---:|
| Actual Non-Crash | 3,834 | 13 |
| Actual Crash | 224 | 887 |

The ensemble achieves high precision in identifying 5-day downside-risk events, while detecting 79.84% of observations belonging to the crash class.

### Trading Strategy Performance

| Metric | Model | Paper |
|---|---:|---:|
| Annualized Return | 21.03% | 40.84% |
| Annualized Volatility | 11.94% | 13.23% |
| Sharpe Ratio | 1.60 | 2.51 |
| Maximum Drawdown | -20.16% | -18.12% |
| Information Ratio | 0.53 | 1.73 |
| CAPM Alpha (Annualized) | 16.53% | 28.00% |
| CAPM Beta | 0.265 | 0.51 |
| Alpha T-statistic | 6.77 | 14.03 |

The strategy produces a 21.03% annualized return with 11.94% annualized volatility and a Sharpe ratio of 1.60. Its CAPM beta of 0.265 indicates substantially lower market exposure than a direct S&P 500 position.

> **Important:** The reported classification and trading results are **in-sample**. The models are evaluated on the same dataset used for training. The analysis also excludes transaction costs. Therefore, these results should not be interpreted as evidence of out-of-sample predictive performance or live trading profitability.

---

## Repository Structure

```text
market-risk-forecasting/
│
├── notebooks/
│   ├── 01_EDA.ipynb
│   └── 02_Modeling.ipynb
│
├── .gitignore
├── README.md
└── requirements.txt
