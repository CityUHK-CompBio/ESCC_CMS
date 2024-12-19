# The consensus molecular subtypes of esophageal squamous cell carcinoma

Esophageal squamous cell carcinoma (ESCC) lacks a standardized classification.  This study presents a novel, robust four-subtype classification system, **ECMS** (ESCC Consensus Molecular Subtypes): **ECMS1-MET** (metabolic), **ECMS2-CLS** (classical), **ECMS3-IM** (immunomodulatory), and **ECMS4-MES** (mesenchymal).  Furthermore, we developed **imECMS**, an image-based classifier using deep learning to assign ESCC patients to these subtypes from H&E images.  ECMS/imECMS subtypes correlate with distinct molecular characteristics, prognoses, and treatment responses, providing a valuable tool for precision medicine in ESCC.

The repository contains the construction of ECMS and the implementation of a user-friendly image classifier(imECMS), including all the code required for generating figures for publication. 

![image](ECMS_2024.png)
## 1.ESCC Consensus Molecular Subtypes（ECMS) Classifier

We built a gene expression data-based classifier for ECMSs prediction. 

### Example: TCGA-ESCC, GSE53625 and GSE45670 prediction

Our study used the TCGA-ESCC, GSE53625, and GSE45670 datasets. Here, we show how to obtain ECMS label using our gene expression classifier.

```{r}
load("./ECMS.model.rdata")

##### TCGA
tcga.predict = predict(rf.cl, tcga.val.df)
table(tcga.predict)

##### GSE53625
gse53625.predict = predict(rf.cl, gse53625.val.df)
table(gse53625.predict)

##### GSE45670
gse45670.predict = predict(rf.cl, gse45670.val.df)
table(gse45670.predict)
```

### Assign patient to ECMSs

If you have your own data, pls prepare a scaled expression matrix first. 

```{r}
# check features used in our model
gene.features

# prepare data.frame
# suppose expr.df is a gene expression matrix.
pre.df = expr.df[gene.features,]

# transpose
pre.df = t(pre.df)

# nomarlization: features are required to be normalized by Z score
pre.df = scale(pre.df)

# prediction
predict(rf.cl, pre.df)
```

## 2.Image Classifier (imECMS)
The imECMS.Rmd provides the code for imECMS classifier.

### How to Use:

Replace trained_model, user_tissue_features, and clinical_data with actual input data and model variables.

You can find all the code for subsequent OS and DFS analysis in imECMS.Rmd, and all the features we used for analysis in Supplementary Table S12 of the paper.
 
```r
# Load the trained model
# load("./final_trained_model.rdata")  # Uncomment and replace with the model file path

# Predict results using the test dataset
test_data <- user_tissue_features  # Input user tissue feature data
prediction_results <- predict(trained_model, test_data)  # Predicted labels
prediction_probabilities <- predict(trained_model, test_data, probability = TRUE)  # Predicted probabilities

# Predict results for core samples
merged_data <- merge(user_core_samples, test_data, by = "row.names", all = FALSE)[, -c(1, 2)]  # Merge and clean data
core_predictions <- predict(trained_model, merged_data)  # Predict core samples

# Majority voting based on patient IDs
sample_names <- substr(rownames(test_data), 1, 9)  # Extract patient IDs
unique_patient_ids <- unique(sample_names)  # Unique patient IDs

voting_results <- sapply(unique_patient_ids, function(patient_id) {
  # Get predictions for current patient
  indices <- which(sample_names == patient_id)
  patient_predictions <- prediction_results[indices]
  freq_table <- table(patient_predictions)  # Frequency table for predictions
  
  # Majority voting logic
  if (sum(freq_table == max(freq_table)) >= 2) {
    result <- NA  # Tie condition
  } else {
    result <- names(freq_table)[which.max(freq_table)]  # Most frequent prediction
  }
  
  # Override condition for ECMS4 presence
  if (freq_table[which.max(freq_table)] != length(indices) && "ECMS4" %in% names(freq_table)) {
    result <- "ECMS4"
  }
  
  return(result)
})

# Match predictions with clinical data
voting_results <- voting_results[match(clinical_data$Patient_ID, names(voting_results))]
final_results_df <- data.frame(Patient_ID = names(voting_results), Predicted_Label = voting_results)

# Display final results
print(final_results_df)
```

Author(Zhu Zhongxu, Zhang Yinghan)
