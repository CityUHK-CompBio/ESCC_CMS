# ESCC_CMS
Esophageal squamous cell carcinoma (ESCC) lacks a standardized classification.  This study presents a novel, robust four-subtype classification system, ECMS (ESCC Consensus Molecular Subtypes): ECMS1-MET (metabolic), ECMS2-CLS (classical), ECMS3-IM (immunomodulatory), and ECMS4-MES (mesenchymal).  Furthermore, we developed imECMS, an image-based classifier using deep learning to assign ESCC patients to these subtypes from H&E images.  ECMS/imECMS subtypes correlate with distinct molecular characteristics, prognoses, and treatment responses, providing a valuable tool for precision medicine in ESCC.

The repository contains the construction of ECMS and the implementation of a user-friendly image classifier(imECMS), including all the code required for generating figures for publication. 

![image](ECMS_2024.png)
## 1.ESCC Consensus Molecular Subtypes（ECMS) Classifier

## 2.Image Classifier (imECMS)
The imECMS.Rmd provides the code for imECMS classifier.

How to Use:
	•	Replace trained_model, user_tissue_features, and clinical_data with actual input data and model variables.
 
```r
# Load the trained model
# load("./final_trained_model.rdata")  # Replace with the path to your model file

# Predict results using the test dataset
test_data <- user_tissue_features  # Input user tissue feature data
prediction_results <- predict(trained_model, test_data)  # Obtain predicted labels
prediction_probabilities <- predict(trained_model, test_data, probability = TRUE)  # Obtain probabilities

# Predict results for core samples
merged_data <- merge(user_core_samples, test_data, by = "row.names", all = FALSE)  # Merge core sample data
merged_data <- merged_data[, -c(1,2)]  # Remove unnecessary columns
core_predictions <- predict(trained_model, merged_data)  # Predict core samples

# Majority voting based on patient IDs
sample_names <- substr(rownames(test_data), 1, 9)  # Extract sample IDs
unique_patient_ids <- unique(sample_names)  # Get unique patient IDs
voting_results <- list()  # Initialize a list to store voting results
frequency_tables <- list()  # Initialize a list to store frequency tables

for (patient_id in unique_patient_ids) {
  indices <- which(sample_names == patient_id)  # Get indices for the current patient
  patient_predictions <- prediction_results[indices[1]:indices[length(indices)]]  # Collect all predictions for the patient
  freq_table <- table(patient_predictions)  # Count the frequency of each prediction
  frequency_tables[[patient_id]] <- freq_table  # Store frequency table
  
  # Majority voting: determine the most frequent label
  if (sum(freq_table == max(freq_table)) >= 2) {
    final_result <- NA  # If there is a tie, set result as NA
  } else {
    final_result <- names(freq_table)[which.max(freq_table)]  # Assign the most frequent label
  }
  
  # Special condition: if ECMS4 is present but not all votes agree, set the result to "ECMS4"
  if (freq_table[which.max(freq_table)] != length(indices) && freq_table[4] > 0) {
    final_result <- "ECMS4"
  }
  
  voting_results[[patient_id]] <- final_result  # Store the final result
}

names(voting_results) <- unique_patient_ids  # Assign names to voting results
voting_results <- do.call(c, voting_results)  # Combine results into a single vector

# Match predictions with clinical data
voting_results <- voting_results[match(clinical_data$Patient_ID, names(voting_results))]
final_results_df <- as.data.frame(voting_results)  # Convert results to a data frame
print(final_results_df)
```

Author(Zhu Zhongxu, Zhang Yinghan)
