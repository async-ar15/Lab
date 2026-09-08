# Phase 02: ML Fundamentals
> *The fat eliminated, the muscle exposed. This is the first-principles breakdown of the classical Machine Learning pillars.*

---

## PILLAR 1: Supervised Learning (The Predictive Engines)

### 1. Linear & Logistic Regression
* **First Principle:** We fit a mathematical equation to observed data to map inputs to a continuous output (Linear) or a probability between 0 and 1 (Logistic).
* **The Muscle:** 
  * **Linear Regression:** Fits a hyperplane. Loss is usually Mean Squared Error (MSE). `y = Wx + b`. It’s the foundational building block for Neural Network layers.
  * **Logistic Regression:** Passes the linear output through a Sigmoid function. Used for binary classification. Loss is Binary Cross-Entropy (Log Loss).

### 2. Decision Trees & SVMs (Support Vector Machines)
* **First Principle:** Instead of fitting a continuous line, we can partition space either by drawing hard boundaries based on feature splits (Trees) or by maximizing the geometric margin between classes (SVM).
* **The Muscle:** 
  * **Decision Trees:** Non-linear by nature. They split data using metrics like Gini Impurity or Information Gain (Entropy). Extremely interpretable but highly prone to overfitting.
  * **SVM:** Finds the hyperplane that has the maximum margin (distance) from the nearest data points (Support Vectors) of both classes. Can use "The Kernel Trick" to project data into infinite dimensions to separate non-linear data without actually computing the infinite dimensions.

### 3. K-Nearest Neighbors (KNN) & Naive Bayes
* **First Principle:** Predictions can be made by looking at the closest historical examples (KNN) or by applying probability theorems assuming features are independent (Naive Bayes).
* **The Muscle:** 
  * **KNN:** A "lazy learning" algorithm. No training phase; inference requires computing distance (Euclidean/Manhattan) to all points in the dataset. Sensitive to outliers and the Curse of Dimensionality.
  * **Naive Bayes:** Pure application of Bayes Theorem. Called "Naive" because it assumes all features are statistically independent, which is rarely true, yet it works shockingly well for text classification and spam filtering.

---

## PILLAR 2: Model Evaluation & The Reality of Data

### 4. Bias-Variance Tradeoff & Evaluation Metrics
* **First Principle:** A model can fail in two ways: it is too simple to capture the underlying truth (High Bias / Underfitting) or it memorizes the noise in the training data instead of the signal (High Variance / Overfitting).
* **The Muscle:** 
  * **The Tradeoff:** As model complexity increases, Bias decreases but Variance increases. We seek the sweet spot.
  * **Metrics:** Accuracy is useless on imbalanced data. We use Precision (out of all predicted positives, how many were real?), Recall (out of all real positives, how many did we find?), F1-Score (harmonic mean), and ROC-AUC (measuring the trade-off between True Positive Rate and False Positive Rate).

### 5. Imbalanced Data & Anomaly Detection
* **First Principle:** In the real world, the thing you are looking for (fraud, disease, defects) is incredibly rare. 
* **The Muscle:** 
  * **Imbalanced Data Tactics:** SMOTE (Synthetic Minority Over-sampling Technique), under-sampling the majority class, or using class weights in the loss function to heavily penalize missing the minority class.
  * **Anomaly Detection:** Instead of binary classification, we model the normal distribution of the data. Anything that falls outside a learned probability threshold (Isolation Forests, One-Class SVM) is flagged as an anomaly.

---

## PILLAR 3: Unsupervised Learning & Feature Engineering

### 6. Unsupervised Learning (Clustering)
* **First Principle:** Sometimes we have data without labels. We must algorithmically discover the underlying structure or groupings within it.
* **The Muscle:** 
  * **K-Means:** Iteratively assigns points to the nearest cluster centroid, then moves the centroid to the mean of those points. Assumes spherical clusters.
  * **DBSCAN:** Density-based clustering. Connects points that are tightly packed together. Can discover clusters of arbitrary shapes and automatically flags outliers as noise.

### 7. Feature Engineering & Selection
* **First Principle:** Garbage In, Garbage Out. The representation of the data is often more important than the algorithm itself.
* **The Muscle:** 
  * **Engineering:** Creating new signals. One-Hot Encoding for categorical variables, Scaling/Standardization for numerical features (so algorithms like KNN or Gradient Descent aren't skewed by large magnitudes).
  * **Selection:** Removing noise. Filter methods (correlation matrices), Wrapper methods (Recursive Feature Elimination), or Embedded methods (L1/Lasso Regularization driving weights to exactly zero).

### 8. Time Series
* **First Principle:** When data is sequential and ordered by time, the assumption of independent data points breaks. We must model trends, seasonality, and autocorrelation.
* **The Muscle:** 
  * Moving Averages, Autoregressive Integrated Moving Average (ARIMA). The target is often not the raw value, but the *difference* between the current value and the previous value (making the series stationary).

---

## PILLAR 4: Orchestration & The State of the Art

### 9. Ensemble Methods
* **First Principle:** The Wisdom of the Crowd. Combining many weak models creates one incredibly strong, robust model.
* **The Muscle:** 
  * **Bagging (Random Forests):** Train many deep Decision Trees in parallel on random subsets of data and features. Averages their predictions to drastically reduce Variance (overfitting).
  * **Boosting (XGBoost, LightGBM):** Train shallow trees sequentially. Each new tree specifically tries to correct the errors (residuals) made by the previous trees. The undisputed kings of tabular data competitions.

### 10. Hyperparameter Tuning & ML Pipelines
* **First Principle:** Code must be reproducible, and models have settings (hyperparameters) that cannot be learned via gradients.
* **The Muscle:** 
  * **Tuning:** Grid Search (exhaustive), Random Search (often better), or Bayesian Optimization (using past trials to predict the next best hyperparameters).
  * **Pipelines:** Chaining Imputers, Scalers, and Models into a single object prevents "Data Leakage" (where test data statistics leak into the training process).
