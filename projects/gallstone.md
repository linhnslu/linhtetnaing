---
layout: default
title: Gallstone Disease Prediction
---

---
title: "Prediction of Gallstone Disease: A Comprehensive Report"
output: html_document
date: "2025-12-3"
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
library(dplyr)
library(knitr)
```

## Overview

Gallstone is a common disease, and its formation is influenced by multiple factors such as age, gender, obesity, and other health conditions.[1] Non-invasive measures such as body composition offer potential for early detection.[2] Identifying individuals at risk of gallstone disease could enable timely clinical interventions and help reduce complications.[2] The gallstone dataset provides demographic, bioimpedence, and laboratory data that are highly relevant for predicting the presence of gallstones. 
The primary goal of the project is to identify patients at risk of gallstones using clinical, laboratory, and bioimpedance data before imaging. Additionally, the project aims to assess which markers most reliably signal gallstone risk so that they could be priortized in screening and risk stratification. 

## Dataset Description

The gallstone dataset is publicly available through the University of California, Irvine (UCI) Machine Learning Repository (donated on April 19, 2025).[3] Data for this clinical dataset was collected from the Internal Medicine Outpatient Clinic at Ankara VM Medical Park Hospital. The dataset contains records from 319 individuals (June 2022–June 2023), with 161 confirmed cases of gallstone disease.[2] A detailed description of all variables is provided here: https://archive.ics.uci.edu/dataset/1150/gallstone-1 

## Methodology

A suite of classification models was developed using repeated 10-fold cross-validation to evaluate predictive performance. The models included Logistic Regression, Linear Discriminant Analysis (LDA), Quadratic Discriminant Analysis (QDA), K-Nearest Neighbors (KNN), Random Forest, Gradient Boosting Machine (GBM), and Support Vector Machines (SVM). Prior to model fitting, data were preprocessed by centering and scaling to ensure comparability across algorithms. All models were trained on the same training subset using standardized tuning and evaluation procedures within the caret framework. Hyperparameter optimization was conducted via grid search.
Predictive performance was primarily evaluated using the Area Under the Receiver Operating Characteristic Curve (AUC-ROC), with secondary consideration for accuracy, sensitivity, and specificity. AUC-ROC evaluates model’s ability to discriminate between patients with and without gallstone disease. whereas sensitivity reflects ability to correctly detect gallstone disease, specificity reflects the ability to correctly identify non-disease patients, and accuracy summarizes the overall correct classification rate. Using both a general performance assessment like AUC-ROC and specific, threshold-dependent metrics, such as sensitivity and specificity allows the selection of models based on prioritized clinical needs. 

## Results and Interpretation

```{r, echo=FALSE}
tab <- tribble(
  ~Rank, ~Model,
  ~Holdout_ROC, ~Holdout_ACC, ~Holdout_SENS, ~Holdout_SPEC,
  ~CV_ROC,      ~CV_ACC,      ~CV_SENS,      ~CV_SPEC,
  1, "LogReg",
  0.907, 0.841, 0.875, 0.806,
  0.827, 0.778, 0.844, 0.712,
  2, "svmLinear",
  0.903, 0.825, 0.906, 0.742,
  0.840, 0.778, 0.849, 0.706,
  3, "RF",
  0.867, 0.810, 0.812, 0.806,
  0.860, 0.783, 0.807, 0.759,
  4, "GBM",
  0.848, 0.762, 0.750, 0.774,
  0.862, 0.787, 0.802, 0.772,
  5, "QDA",
  0.902, 0.873, 0.938, 0.806,
  0.845, 0.787, 0.909, 0.663,
  6, "LDA",
  0.892, 0.825, 0.844, 0.806,
  0.792, 0.740, 0.839, 0.638,
  7, "svmRadial",
  0.856, 0.730, 0.688, 0.774,
  0.818, 0.766, 0.803, 0.738,
  8, "KNN",
  0.822, 0.730, 0.844, 0.613,
  0.757, 0.660, 0.849, 0.467
)

# Print nicely (e.g. in R Markdown)
kable(
  tab,
  caption = "Table 1: Model performance rankings based on holdout and cross-validation evaluations"
)
```
<small><i>ACC = Accuracy, ROC = Area Under the Receiver Operating Characteristic Curve (AUC-ROC), SPEC = Specificity, SENS = Sensitivity</i></small>
<small><i>GBM = Gradient Boosting Machine, KNN = k-Nearest Neighbors, LDA = Linear Discriminant Analysis, LogReg = Logistic Regression, QDA = Quadratic Discriminant Analysis, RF = Random Forest, svmLinear = Support Vector Machine (Linear Kernel), svmRadial = Support Vector Machine (Radial Kernel)</i></small>

```{r my-image-chunk, echo=FALSE, out.width="70%"}
knitr::include_graphics("cv_performance_metrics.png")
```

Across the resampled training evaluations, the GBM model demonstrated the strongest overall performance. It achieved the highest ROC (0.862), the highest accuracy (0.787), the highest specificity (0.772), and strong sensitivity (0.802), with the most stable range of performance metrics in cross-validation. Random Forest closely followed, with a comparable ROC (0.860) and balanced performance metrics. QDA achieved the highest sensitivity (0.909), but at the cost of markedly lower specificity (0.663). Logistic regression with lasso regularization achieved a comparable ROC (0.827) and accuracy (0.778), with moderate and balanced sensitivity and specificity. SVM-Linear and SVM-Radial also performed well, with moderate ROC, accuracy, sensitivity, and specificity. KNN performed the worst in training, with the lowest ROC and poor specificity. Taken together, GBM demonstrated the best combination of discrimination, specificity, and consistency in resampling. The baseline model, logistic regression with lasso regularization, performed solidly and stably with balanced metrics.

Model performance was further assessed using the holdout set. On the holdout set, logistic regression with lasso regularization achieved the highest ROC (0.907), followed closely by SVM Linear (0.903), QDA (0.902), and LDA (0.892). QDA achieved the highest overall accuracy (0.873), logistic regression also performed strongly with accuracy of 0.841. Notably, the ranking shifted on the holdout set compared to cross-validation, with logistic regression, QDA, SVM Linear, and LDA outperforming GBM and RF in terms of ROC and accuracy. Both logistic regression and QDA exhibited the most balanced performance across all metrics on the holdout set. Although SVM Linear achieved comparable ROC and accuracy, its specificity (0.742) was much lower compared to the baseline logistic regression model. GBM, which was the strongest performer during cross-validation, underperformed on the holdout set with lower ROC (0.848) and accuracy (0.774), suggesting possible overfitting.

When both the stability of the CV results and the generalizability reflected by the holdout set performance are considered, the logistic regression model stands out as the most reliable choice for predicting gallstone disease in this dataset. It demonstrated strong discrimination, balanced sensitivity and specificity, and consistent performance across both evaluation approaches.

```{r, echo = FALSE}

tab2 <- tribble(
  ~Variable,       ~LogReg, ~LDA,   ~svmLinear, ~RF,    ~GBM,   ~Average,
  "crp",           100.00,  27.99,  100.00,     100.00, 100.00, 85.60,
  "vit_d",         39.33,   51.86,  27.64,      59.34,  47.97,  45.23,
  "hfa3",          82.10,   100.00, 14.47,      3.22,   6.07,   41.17,
  "protein",       53.08,   43.11,  39.58,      33.38,  34.21,  40.67,
  "ecf_tbw_ratio", 49.17,   51.89,  36.78,      37.26,  24.66,  39.95
)

kable(
  tab2,
  caption = "Table 2: Top five predictors ranked by aggregated scaled importance across models"
)
```
<small><i>GBM = Gradient Boosting Machine, LDA = Linear Discriminant Analysis, LogReg = Logistic Regression, 
RF = Random Forest, svmLinear = Support Vector Machine (Linear Kernel)</i></small>
<small><i>crp = C-reactive protein, vit_d = Vitamin D, hfa = Hepatic Fat Accumulation, protein = Body Protein Content, ecf_tbw_ratio = Extracellular Fluid to Total Body Water Ratio</i></small>

## Conclusion

The results suggest that the inflammatory marker CRP is the dominant signal for gallstone risk, supported by consistently high variable-importance scores across interpretable and non-interpretable models alike. Metabolic factors such as vitamin D and hepatic fat accumulation, along with bioimpedence-derived measures such as body protein content, and ECF/TBW ratio also contributed significantly to the prediction of gallstone disease, athough their influence varied by modeling technique. This has direct implications for clinical decision-making: targeted laboratory assessments (CRP, lipid profile, vitamin D) and the potential integration of bioimpedance measurements may enhance screening and risk-stratification strategies.

Given its strong generalization, consistency across different evaluations, and inherent transparency and interpretability, logistic regression with lasso regularization is the most appropriate choice for deployment in decision support, where clinicians must understand and trust why a patient is identified as high-risk.


---

## 🔗 Code Repository
👉 https://github.com/yourusername/gallstone-prediction

[← Back to Home](/index.html)

