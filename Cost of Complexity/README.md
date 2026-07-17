# **🔄 The Cost of Complexity: Linear Models vs. Tree Ensembles on Structured Data**

- **The Core Idea:** Take a classic tabular dataset (e.g., housing prices or credit risk) and benchmark simple, highly interpretable models against complex ensemble methods.
- **The Research Question:** At what point does the marginal increase in predictive accuracy (e.g., switching from Linear Regression to a Random Forest) fail to justify the loss of model interpretability and increased computational cost?

- We are testing whether we actually need super complicated computer models to make good predictions. We will take a basic, easy-to-understand model and compare it to a complex, 'black-box' model on the same data. If the basic one is almost as accurate, it’s probably better to use because anyone can understand how it works

By testing this hypothesis on exactly **one regression task** (predicting a continuous number) and **one classification task** (predicting a category), the findings will be much more robust and convincing to academic readers

## **Dataset 1 (Regression): Ames Housing Dataset**

* **The Goal:** Predict the continuous sale price of a house
* **Kaggle Link:** [Ames Housing Dataset on Kaggle](https://www.kaggle.com/datasets/shashanknecrothapa/ames-housing-dataset)

> **Why this dataset is a good choice :** This dataset contains a rich mix of continuous, ordinal, and categorical features, making it the perfect playground to see if a complex Random Forest or XGBoost can actually outperform a carefully tuned Lasso or Ridge Regression once you account for the effort of feature engineering

## **Dataset 2 (Classification): German Credit Risk Dataset**

* **The Goal:** Classify a loan applicant as a "good" or "bad" credit risk (binary classification)
* **Kaggle Link:** [German Credit Dataset on Kaggle](https://www.kaggle.com/datasets/jumpingdino/german-credit-dataset)

> **Why this dataset is a good choice :** In credit scoring, explaining *why* a loan was denied is a legal and ethical requirement, so comparing a simple Logistic Regression (highly interpretable) against a complex Gradient Boosting Machine (black-box) directly addresses your research question regarding the trade-off between a tiny boost in accuracy and the loss of explainability

### **How These Two Build Your Paper's Narrative**

* **The Baselines (Linear/Interpretable Models):** There would be use of **Multiple Linear Regression (with L1/L2 regularization)** for the housing data and **Logistic Regression** for the credit risk data
* **The Challengers (Tree Ensembles):** One is able to run **Random Forests** and **XGBoost/LightGBM** on both datasets
* **The Complexity Evaluation:** The **Performance (R² or AUC-ROC)** and **Compute Time/Model Size** can be easily plotted, visualizing exactly where the "diminishing returns" curve flattens out

# **📊 The Experimental Results & Core Findings**

We completed the exploratory data analysis (EDA) and ran all our baseline and challenger models across both datasets. The empirical evidence heavily supports our core idea: complex models are often not worth the trade-off on structured data.

### **1. Performance vs. Complexity (Training Speed)**

First, we mapped how much extra computer processing time we have to spend just to get a tiny increase (or sometimes even a decrease!) in score performance.

* **Ames Housing (Regression):** Gradient Boosting gave us an $R^2$ score of **0.8993**, while Ridge Regression gave us **0.8907**. We only gained a microscopic **0.86% accuracy boost**—meaning Gradient Boosting only saved us about $1,742 on an average house prediction—but it cost us a massive jump in training time.
* **German Credit Risk (Classification):** Standard Logistic Regression and Elastic Net completely beat out the complex tree ensembles. The simple models got an **ROC-AUC score of 0.809**, while the heavy XGBoost model dragged performance all the way down to **0.772**.

> **The Takeaway:** Complex models do not automatically mean better results. For credit data, the simpler models were actually more accurate, and for housing data, the extra accuracy gained from boosting was practically negligible in the real world.

| Ames Housing - Complexity vs. $R^2$ | German Credit Risk - Complexity vs. AUC-ROC |
| :---: | :---: |
| <img width="863" alt="Ames Housing Complexity" src="https://github.com/user-attachments/assets/941a87a5-bc34-4e11-ae78-983cb505bd19" /> | <img width="865" alt="Credit Risk Complexity" src="https://github.com/user-attachments/assets/ff2d1150-635c-4764-8fe8-e883e30ad39b" /> |

---

### **2. The "Explainability Gap" (What We Lose)**

To understand the human cost of choosing a complicated model, we plotted the feature importances side-by-side to see what happens to our ability to audit the models.

| Ames Housing Explainability Gap | German Credit Risk Explainability Gap |
| :---: | :---: |
| <img width="1384" alt="Ames Explainability Gap" src="https://github.com/user-attachments/assets/5787d78d-d547-4905-9cca-f17e45d9ea8e" /> | <img width="1436" alt="Credit Risk Explainability Gap" src="https://github.com/user-attachments/assets/5383c1af-ed75-412f-bd2b-c3d2e56c6050" /> |

* **The Linear Advantage (Auditable):** On the left side of the charts, our linear and logistic models give us clean, signed coefficients. Anyone can easily read the chart and say exactly *how* a feature affects the outcome (e.g., seeing exactly how much a specific neighborhood raises a house price, or how a specific credit account status drops bad credit odds).
* **The Tree Disadvantage (Unsigned MDI):** On the right side, the Random Forest only tells us that a feature is "important" via Mean Decrease in Impurity (MDI). It gives us a flat magnitude but completely hides the direction. We cannot tell if a feature pushes a prediction up or down without using extra, complicated mathematical explanation tools.

---

### **3. Deep-Dive Error Analysis**

We looked deeper into the actual mistakes the models were making to see if Gradient Boosting was actually discovering special hidden patterns that Ridge Regression missed.

| Ames Housing - Residual Analysis | German Credit Risk - ROC Curves |
| :---: | :---: |
| <img width="848" alt="Ames Residual Analysis" src="https://github.com/user-attachments/assets/cae26544-8040-44cd-bf0c-ddac006cac66" /> | <img width="783" alt="Credit Risk ROC Curves" src="https://github.com/user-attachments/assets/a49a9fec-b39c-4155-bf9f-c30d0c5cdc81" /> |

* **Ames Housing Overlap:** The correlation between the absolute errors of Ridge and Gradient Boosting is incredibly high at **0.888**. Their residual error distributions overlap almost perfectly. This proves mathematically that Gradient Boosting isn't finding a fundamentally different structural pattern; it is making the exact same mistakes as the simple linear model and just shaving off tiny numbers on the extreme margins.
* **Credit Risk ROC Domination:** Looking at the ROC curves, the Logistic Regression line consistently hugs the top-left corner much better than XGBoost. Because this tabular data is dominated by clean linear structures, tree models over-segment the data space, resulting in overfitting that actually harms their ability to generalize to unseen test data.
