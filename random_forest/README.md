# Random Forrest Model:

In this step, we train a Random Forest classifier using the ranger package to predict whether a loan is Fully Paid or Charged Off. We use 200 trees and enable permutation-based variable importance to identify which features contribute most to the prediction.

What is a Random Forrest? 

A random forrest curates individual trees, in this case within the example we are creating 200 different trees

```
rfModel1 <- ranger(loan_status ~., data=lcdfTrn %>%
select(-all_of(varsOmit)),
num.trees = 200,
importance='permutation',
probability = TRUE)

```

- loan_status ~ .: predicts loan status using all remaining variables
- varsOmit: excludes columns that could cause data leakage or are not relevant
- probability = TRUE: enables probability output for ROC/AUC evaluation
- importance = 'permutation': calculates feature importance by how much model performance drops when a variable is shuffled


### Model Results 

```
Ranger result

Call:
 ranger(loan_status ~ ., data = lcdfTrn %>% select(-all_of(varsOmit)),      num.trees = 200, importance = "permutation", probability = TRUE) 

Type:                             Probability estimation 
Number of trees:                  200 
Sample size:                      69909 
Number of independent variables:  45 
Mtry:                             6  
Target node size:                 10 
Variable importance mode:         permutation 
Splitrule:                        gini 
OOB prediction error (Brier s.):  0.02493399

```

- **Mtry**: Choosing 6 random variables and determing which is giving the best split using the gini split method
- **Target Node Size:** Each node needs to have at least 10 observation
- **Variable Importance Mode --> Permutation**: 
The permutation importance method works by randomly shuffling the values of a single predictor variable across the dataset and measuring how much the model’s performance decreases as a result. If shuffling a variable's values causes a large increase in prediction error (e.g., Brier score or misclassification rate), then the variable is considered important — because the model relied heavily on that variable to make accurate predictions. If shuffling has little to no effect, then the model didn’t depend much on that variable, so it's considered less important.
- **Gini**: The idea is to create splits that create "pure" nodes, aka, nodes with majority classes.
-OOB Prediction Error (Brier Score): Random Forest models use an internal cross-validation method called **Out-of-Bag (OOB) estimation:** Each tree is trained on a bootstrap sample (roughly 70% of the data, sampled with replacement), and the remaining ~30% of data — not seen by the tree — is used to evaluate the tree’s prediction. The model aggregates these OOB predictions across all trees to estimate generalization error.
In this case, the OOB error is reported as a Brier score of 0.0249, which is a measure of how close the predicted probabilities are to the true outcomes. A lower Brier score indicates better calibrated probabilistic predictions — and 0.0249 is considered quite low, suggesting strong model performance.
### Variable Importance 

```

vimp_rfGp <- importance(rfModel1)
View(vimp_rfGp)

            loan_amnt            funded_amnt        funded_amnt_inv 
          0.0475466989           0.0495694156           0.0505143078 
                  term               int_rate                  grade 
          0.0000000000           0.0111629970           0.0067124128 
             sub_grade             emp_length         home_ownership 
          0.0102313391           0.0003594914           0.0010432047 
            annual_inc    verification_status                issue_d 
          0.0030280424           0.0007049615           0.0197104900 
            pymnt_plan                purpose             addr_state 
          0.0000000000           0.0004362172           0.0002821515 
                   dti         inq_last_6mths mths_since_last_delinq 
          0.0013485734           0.0005392438           0.0001233885 
             revol_bal             revol_util              total_acc 
          0.0028221671           0.0015408650           0.0012716293 
   initial_list_status        total_pymnt_inv          total_rec_int 
          0.0003158548           0.1659215657           0.0301363983 
          last_pymnt_d        last_pymnt_amnt       application_type 
          0.0516467138           0.0794494904           0.0000729172 
           tot_cur_bal       total_rev_hi_lim   acc_open_past_24mths 
          0.0051233890           0.0040202211           0.0025568967 
           avg_cur_bal         bc_open_to_buy                bc_util 
          0.0046784502           0.0018355724           0.0011326480 
    mo_sin_old_il_acct   mo_sin_old_rev_tl_op  mo_sin_rcnt_rev_tl_op 
          0.0004065643           0.0010161957           0.0010359977 
        mo_sin_rcnt_tl               mort_acc   mths_since_recent_bc 
          0.0012978112           0.0011856929           0.0006248400 
 mths_since_recent_inq     num_tl_op_past_12m        tot_hi_cred_lim 
          0.0003975494           0.0016640642           0.0055625396 
        total_bc_limit          hardship_flag    disbursement_method 
          0.0037307227           0.0000000000           0.0000000000 

```

This reveals which variables most influence the prediction. Higher importance values indicate stronger predictive power. I can see that loan amount, funded amount, funded_amnt_inv, term, and interest rate are important variables correlated in accuraately predicting if a loan will be paid or defaulted. 



Training Predictions and Classification Performance

```
scoreTrn <- predict(rfModel1, lcdfTrn)
head(scoreTrn$predictions)
     Fully Paid Charged Off
[1,]  0.9907222 0.009277778
[2,]  0.9880278 0.011972222
[3,]  0.9627679 0.037232143
[4,]  0.9926190 0.007380952
[5,]  0.9876190 0.012380952
[6,]  0.9531944 0.046805556
```

We extract the predicted probabilities for each class (Fully Paid and Charged Off) on the training set.

```
pred_label <- factor(ifelse(scoreTrn$predictions[,"Fully Paid"] > 0.7, "Fully Paid", "Charged Off"))
table(Prediction = pred_label, Actual = lcdfTrn$loan_status)

confusionMatrix(pred_label , lcdfTrn$loan_status)

Confusion Matrix and Statistics

             Reference
Prediction    Fully Paid Charged Off
  Fully Paid       60213           0
  Charged Off          7        9689
                                     
               Accuracy : 0.9999     
                 95% CI : (0.9998, 1)
    No Information Rate : 0.8614     
    P-Value [Acc > NIR] : < 2e-16    
                                     
                  Kappa : 0.9996     
                                     
 Mcnemar's Test P-Value : 0.02334    
                                     
            Sensitivity : 0.9999     
            Specificity : 1.0000     
         Pos Pred Value : 1.0000     
         Neg Pred Value : 0.9993     
             Prevalence : 0.8614     
         Detection Rate : 0.8613     
   Detection Prevalence : 0.8613     
      Balanced Accuracy : 0.9999  


```

This final step shows a confusion matrix at a custom threshold of 0.7 for the “Fully Paid” class, meaning we only predict a loan as “Fully Paid” if its probability exceeds 70%. This allows us to analyze model performance at a specific confidence level, which is useful in risk-sensitive applications like loan approvals.


### Testing Data Set Validation 

```
RFscoreTest = predict(rfModel1,lcdfTest)

RF_pred_tst_label = factor(ifelse(RFscoreTest$predictions[,"Fully Paid"] > 0.7, "Fully Paid","Charged Off"))

confusionMatrix(RF_pred_tst_label, lcdfTest$loan_status)

Confusion Matrix and Statistics

             Reference
Prediction    Fully Paid Charged Off
  Fully Paid       25708         363
  Charged Off        253        3726
                                          
               Accuracy : 0.9795          
                 95% CI : (0.9778, 0.9811)
    No Information Rate : 0.8639          
    P-Value [Acc > NIR] : < 2.2e-16       
                                          
                  Kappa : 0.9118          
                                          
 Mcnemar's Test P-Value : 1.124e-05       
                                          
            Sensitivity : 0.9903          
            Specificity : 0.9112          
         Pos Pred Value : 0.9861          
         Neg Pred Value : 0.9364          
             Prevalence : 0.8639          
         Detection Rate : 0.8555          
   Detection Prevalence : 0.8676          
      Balanced Accuracy : 0.9507          
                                          
       'Positive' Class : Fully Paid    

```

I won't go into the terminalogy like before, but this model is very strong having high accuracy as well as a high Sensitivity and Specificity! This is just a comparison point to compare how a random forrest model compares to our decision tree model before, but next we are going create a random forrest model where actual returns are the target variable and see if we are able to predict an investors return on each loan. 


### Random Forrest Model for Actual Returns 

```

rfModel_Ret <- ranger(actualReturn ~.,
data=subset(lcdfTrn,
select=-c(annRet, actualTerm, loan_status)),
num.trees =200,
importance='permutation')


```

This is not a classifcation result anymore, in the past we wanted to find out whether a loan will be paid off or if the loan will default. We are now asking the model to return to us the actual returns for each loan, "actualReturns" is our target variable. As a refresher, actualReturns was calculated using the code below

```
lcdf1$actualReturn <- ifelse(lcdf1$actualTerm>0, ((lcdf1$total_pymnt -lcdf1$funded_amnt)/lcdf1$funded_amnt)*(1/lcdf1$actualTerm)*100, 0)

Recap: If the loan term is greater than zero, calculate the return as the difference between the amount repaid by the borrower and the original loan amount, divide that differnce by the original loan amount. To express this as an annualized percentage return, divide the result by the loan term, multiply by the original loan amount, and then multiply by 100 to convert it into a percentage. Anythign else the returns are 0.

```

So now this model is telling us how much return on investment we would get based on x amount funded in an anually percentage form. 


```
# Plot the actual vs. predicted values
plot(rfPredRet_trn$predictions, lcdfTrn$actualReturn,
     xlab = "Predicted Return",
     ylab = "Actual Return",
     main = "Predicted vs Actual Loan Returns (Testing Set)")

```



