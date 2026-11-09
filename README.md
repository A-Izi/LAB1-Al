# M115 Lab 1: Phishing URL Detection

Lab notebook for Module M115 (AI & Intelligent Systems), Master in Intelligent Systems and Cyber Defense, ENSIASD Taroudant.

[`M115_Lab1_Template.ipynb`](LAB1-ALI.ipynb) builds an interpretable phishing URL classifier with logistic regression, explains it with SHAP, and turns its scores into a SOC alerting policy.

## Dataset

[PhiUSIIL Phishing URL Dataset](https://www.kaggle.com/datasets/kaggleprollc/phishing-url-websites-dataset-phiusiil/data): 235,795 URLs and 56 columns, with no missing values or duplicate rows.

The raw `label` column uses `1` for legitimate, so the notebook creates `IsPhishing = 1 - label` to make the attack the positive class (42.81% of URLs).

## Contents

| Part | Exercises | Topic |
| --- | --- | --- |
| 1. EDA | 1–3 | Data quality, target alignment, distributions, feature correlations |
| 2. Preprocessing | 4 | Stratified 75/25 split, `StandardScaler` fitted on training data only |
| 3. Logistic regression | 5–7 | Five configurations, L1 regularization path, weights and odds ratios |
| 4. Explainability and thresholds | 8–10 | Global and local SHAP plots, decision thresholds of 0.20, 0.50 and 0.85 |

The model uses 49 features. Raw text identifiers and the pre-computed `URLSimilarityIndex` are dropped.

## Results

Test set of 58,949 URLs, at a decision threshold of 0.50:

| Model | Precision | Recall | F1 | False positives | False negatives | Zeroed weights |
| --- | --- | --- | --- | --- | --- | --- |
| M1: Unregularized | 0.9996 | 0.9990 | 0.9993 | 11 | 25 | 0 / 49 |
| M2: L2, C=0.001 | 0.9985 | 0.9958 | 0.9971 | 39 | 105 | 0 / 49 |
| M3: L2, C=1.0 | 0.9996 | 0.9989 | 0.9992 | 10 | 28 | 0 / 49 |
| M4: L1, C=0.01 | 0.9993 | 0.9971 | 0.9982 | 18 | 72 | 24 / 49 |
| M5: L2, balanced | 0.9995 | 0.9990 | 0.9992 | 13 | 25 | 0 / 49 |

L1 regularization path: the number of retained features grows from 18 at C=0.002 to 25 at C=0.01, 33 at C=0.1 and all 49 at C=1.0.

Strongest M3 features, with odds ratios per standard deviation:

- Toward phishing: `LetterRatioInURL` (112.49), `DegitRatioInURL` (36.81), `SpacialCharRatioInURL` (36.53).
- Toward legitimate: `IsHTTPS` (0.0010), `NoOfJS` (0.0259), `NoOfSelfRef` (0.0439).

M3 at each decision threshold, on the full test set:

| Threshold | Mode | Precision | Recall | F1 | False positives | False negatives |
| --- | --- | --- | --- | --- | --- | --- |
| 0.20 | Threat hunting | 0.9988 | 0.9994 | 0.9991 | 31 | 15 |
| 0.50 | Default | 0.9996 | 0.9989 | 0.9992 | 10 | 28 |
| 0.85 | Automated blocking | 1.0000 | 0.9975 | 0.9987 | 0 | 64 |

Recommended policy, based on M3:

- Block automatically when P(Phishing) ≥ 0.85 (0 false positives on the test set).
- Send scores from 0.20 up to 0.85 to an analyst (80 URLs, 0.14% of the test set).
- Allow scores below 0.20, with logging (15 phishing URLs pass).

## How to run

1. Place `PhiUSIIL_Phishing_URL_Dataset.csv` in the same folder as the notebook.
2. Install the dependencies:

   ```bash
   pip install numpy pandas matplotlib seaborn scikit-learn shap jupyter
   ```

3. Open the notebook and run all cells. The random seed is fixed at 42.

With scikit-learn 1.8 or later, the `penalty` argument triggers deprecation warnings. These do not stop the notebook from running.
