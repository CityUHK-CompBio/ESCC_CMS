# ESCC_CMS
Esophageal squamous cell carcinoma (ESCC) lacks a standardized classification.  This study presents a novel, robust four-subtype classification system, ECMS (ESCC Consensus Molecular Subtypes): ECMS1-MET (metabolic), ECMS2-CLS (classical), ECMS3-IM (immunomodulatory), and ECMS4-MES (mesenchymal).  Furthermore, we developed imECMS, an image-based classifier using deep learning to assign ESCC patients to these subtypes from H&E images.  ECMS/imECMS subtypes correlate with distinct molecular characteristics, prognoses, and treatment responses, providing a valuable tool for precision medicine in ESCC.

The repository contains the construction of ECMS and the implementation of a user-friendly image classifier(imECMS), including all the code required for generating figures for publication. 

![image](ECMS_2024.png)
## 1.ESCC Consensus Molecular Subtypes（ECMS) Classifier

## 2.Image Classifier (imECMS)
The imECMS.Rmd provides the code for imECMS classifier.

```r
#load("./rst_final_model.rdata")
xtest <- SXMI_tissueOmics_multi_loc
Pred_SXMI <- predict(final_model,xtest)
pred_SXMI_prob <- predict(final_model,xtest, probability=TRUE)

merged_df <- merge(temp1, SXMI_tissueOmics_multi_loc, by = "row.names", all = FALSE)
merged_df <- merged_df[, -c(1,2)]
Pred_SXMI_core <- predict(final_model,merged_df)

Names <- substr(rownames(SXMI_tissueOmics_multi_loc),1,9)
uniqueName <- unique(Names)
Vote_max_slides_SXMI <- list()
frequency_table_list <- list()

for (xxx in uniqueName) {
  ind <- which(Names == xxx)  
  all_result <- Pred_SXMI[ind[1]:ind[length(ind)]]
  frequency_table <- table(all_result)
  frequency_table_list[[xxx]] <- frequency_table
  if (sum(frequency_table == max(frequency_table)) >= 2) {
    back <- NA
  }else {
    most_frequent_element <- names(frequency_table)[which.max(frequency_table)]
    back <- most_frequent_element
  }
  if (frequency_table[which.max(frequency_table)] != length(ind) && frequency_table[4] > 0) {
    back <- "ECMS4"
  }
  Vote_max_slides_SXMI[[xxx]] <- back  # 将结果存储在列表中
}

names(Vote_max_slides_SXMI) <- uniqueName
Vote_max_slides_SXMI <- do.call(c,Vote_max_slides_SXMI)

Vote_max_slides_SXMI <- Vote_max_slides_SXMI[match(SXMI_Clinical$IMID,names(Vote_max_slides_SXMI))]
test=as.data.frame(Vote_max_slides_SXMI)
labels <- factor(Vote_max_slides_SXMI)
legend.labs <- as.vector(na.omit(unique(labels)))

input <- as.data.frame(cbind(SXMI_Clinical$os.time/30,SXMI_Clinical$os.event)) 
input$V1 <- as.numeric(input$V1)
input$V1 <- as.numeric(input$V1)
SXMI_OS <- myplot(input,labels,ylab="Overall survival",font = "sans",
                  risk.table = T,risk.table.ratio = 0.4,title = "SXM-I",
                  legend.pos = c(0.65,0.18),xlab="Follow up",color=c("#00468BFF", "#ED0000FF", "#42B540FF", "#0099B4FF","black"))

input <- as.data.frame( cbind(SXMI_Clinical$rfs.time/30,SXMI_Clinical$rfs.event)) 
input$V1 <- as.numeric(input$V1)
SXMI_DFS <- myplot(input,labels,ylab="Disease-free survival",font = "sans",
                  risk.table = T,risk.table.ratio = 0.4,title = "SXM-I",
                  legend.pos = c(0.65,0.18),xlab="Follow up",color=c("#00468BFF", "#ED0000FF", "#42B540FF", "#0099B4FF"))
```
Author(Zhu Zhongxu, Zhang Yinghan)
