 # Decision Tree Modeling for Loan Status Prediction Using the rpart Package

### Creating a Training and Testing Split 

```
set.seed(1234) # we want the same results every time, the numbers don't matter

indexSet = sample(2,nrow(lcdf2), # creating 2 groups here from the lcdf2 data set 
                  replace = T, # assignment is random for either group any value can go into either group
                  prob = c(0.7,0.3)) # 70% chance of picking 1 and 30% of picking 2

lcdfTrn = lcdf2[indexSet == 1,]
lcdfTest = lcdf2[indexSet == 2, ]

```

Here I am randomly assign each of the 10,000 rows in lcdf2 to either group 1 (training) or group 2 (testing), where each row has a 70% chance of going into group 1 and a 30% chance of going into group 2. The total count for each group will approximately, but not always exactly, follow that 70/30 split. But now we have a training set that we can use as the testing data for the model as well as a testing set to validate our model and see if it is good or bad. 


### Taking Variables Out 

The very last house cleaning item before creating this model we'll want to do is take some variables out of the model manually without taking them out of the data set completely. We'll take out a variable either because;

1. It has too many unique values slowing down the model such as "X" which is the ID
2. Not immportant to the model, we'll determine this by calculating the AUC value for the variables
3. Data Leakage, we don't want to include variables that essentially tell the model the outcome. Example would be things such as collection_recovery, if there are collection_recovery fees associated with a loan, that means it has defaulted.

### Calculating the AUC Value 

Why is the AUC Value Important? 
  - AUC value tell is how well something (variable or model) does in being able to distinguish between classes. The threshold is 50%, if that something is below 50% that means it is worse than guessing if it is higher than 50% that means it is better than guessing

Example 

```
auc(response=lcdf2$loan_status, lcdf2$loan_amnt)
Setting levels: control = Charged Off, case = Fully Paid
Setting direction: controls < cases
Area under the curve: 0.5212
Area under the curve: 0.5212
```

We can interpret this AUC Value as - if you randomly select one fully paid loan and one charged-off loan, there’s a 52.12% chance that the loan with the higher loan amount will be the fully paid one. We can see that the case is equal to Fully Paid and control = Charged Off, so the function is checking if higher values favor the case Fully Paid. If the AUC value is below in this case, it means that higher loan amounts are leaning towards being charged off. This variable is basically just guessing so it is not super important to include within the model. 


```
aucsNum = sapply(lcdf2 %>% select_if(is.numeric), auc, response=lcdf1$loan_status) # this is doing the same thing as it did in the example, but here we are running all the numerical columns using the select_if function to find the auc value and loan status is what we're trying to find out

aucAll<- sapply(lcdf1 %>% 
                    mutate_if(is.factor, as.numeric) %>% # changing factor variables to numeric 
                    select_if(is.numeric), auc, response=lcdf1$loan_status)  # same as the line above
library(broom)

tidy(aucAll) %>% 
    arrange(desc(aucAll)) %>%
    View()
```


![image](https://github.com/user-attachments/assets/9d514908-47f5-407e-8483-9d9a406f4d46)

Here we can see the AUC value for each variable, we can see for example 5 variables are very high. But actualReturn would probably be categorized as a data leakage esque variable  considering there can be negative actual returns letting the model know it was charged off more than likely! We can also see recovery_collection_fee which an activtiy that would only happen if a loan defaults.


Here are the variables we are going to omit from the model that I will store in a variable called varsOmit. So instead of typing out each variable, I can just input this into the model itself.

```
varsOmit <- c(
  'actualTerm', 'actualReturn', 'annRet', 'total_pymnt', 'open_acc', 'num_sats', 
  'num_rev_accts', 'installment', 'X', 'out_prncp', 
  'out_prncp_inv', 'policy_code', 'num_tl_120dpd_2m', 'delinq_amnt', 
  'chargeoff_within_12_mths', 'num_tl_30dpd', 'acc_now_delinq', 'tax_liens', 
  'collections_12_mths_ex_med', 'num_bc_sats', 'num_tl_90g_dpd_24m', 
  'delinq_2yrs', 'pub_rec_bankruptcies', 'tot_coll_amt', 'num_accts_ever_120_pd', 
  'pub_rec', 'num_actv_bc_tl', 'num_rev_tl_bal_gt_0', 'num_actv_rev_tl', 
  'percent_bc_gt_75', 'total_rec_late_fee', 'num_il_tl', 'total_il_high_credit_limit', 
  'pct_tl_nvr_dlq', 'num_bc_tl', 'total_bal_ex_mort', 'num_op_rev_tl','total_rec_prncp','total_rec_prncp', 'collection_recovery_fee','debt_settlement_flag','recoveries','emp_title','title','zip_code','earliest_cr_line','last_credit_pull_d'
)



```

Next we are going to convert the loan_status variable which is the target variable to a factor instead of a character. 

```
lcdf2$loan_status <- factor(lcdf2$loan_status, levels=c("Fully Paid", "Charged Off"))

```


### Model Time 

**First Model**

```
lcDT1a <- rpart(loan_status ~., data=lcdfTrn %>% select(-all_of(varsOmit)), method="class", parms = list(split = "information"), control = rpart.control(minsplit = 30))

printcp(lcDT1a)


lcDT1$variable.importance

```

Let's take this model line by line quickly go over key terms that will continue to pop up!

- rpart: Recursive Partion, this is being used from the rpart package and is creating a decision tree for us to use to determine if a loan will either be "charged off" or "fully paid"!
- Loan_status is the target variable so this is the first input
- ~. this means we are using all other variables as predictors
- %>% piping
- data = lcdfTrn, if we look back to last piece of code we can see this is the training set we created from the original cleaned data set which is called "lcdf2"
- select(-all_if(varsOmit)), takes the training dataset lcdfTrn2, and remove the columns listed in varsOmit before using it to train the model.
- parms = parameters
- split = "information" tells the decision tree to choose splits based on information gain, which looks for the variable and value that best reduces uncertainty about the target (like loan status). It's a solid, commonly used method for classification problems.
- minsplit = 30, means a node must have at least 30 observations in it before it can be split. This helps control how complex the tree becomes and protects against overfitting by avoiding splits based on very small groups of data.


### Model 1 Results

```
print(lcDT1a)

Classification tree:
rpart(formula = loan_status ~ ., data = lcdfTrn %>% select(-all_of(varsOmit)), 
    method = "class", parms = list(split = "information"), control = rpart.control(minsplit = 30))

Variables actually used in tree construction:
[1] issue_d         last_pymnt_amnt last_pymnt_d    loan_amnt      
[5] total_rec_int  

Root node error: 9738/70044 = 0.13903

n= 70044 

         CP nsplit rel error  xerror      xstd
1  0.125009      0   1.00000 1.00000 0.0094029
2  0.092524      3   0.62497 0.62641 0.0076632
3  0.031423      4   0.53245 0.53420 0.0071262
4  0.020538      5   0.50103 0.50288 0.0069304
5  0.018587      8   0.43941 0.46478 0.0066816
6  0.015506      9   0.42083 0.43797 0.0064990
7  0.012220     10   0.40532 0.41230 0.0063176
8  0.011604     11   0.39310 0.40491 0.0062642
9  0.010885     12   0.38150 0.39998 0.0062282
10 0.010000     13   0.37061 0.38889 0.0061462

```

Let's interpret these results 

1. Classiciation Trees: Just reiterating the model that we area using, all the parameters were as mentioned in the code listed before it.
2. Variables Actually Used in Tree Construction: The only variables that the model actually used within the model that produced any meaningful insight within the parameters we established  such as minsplit = 30 and type of split method we are using.
3. Root Node Error: This portion is basically assuming that is we just labeled each value in the dataset as the majority class, which in this case would be "Fully Paid" what percentage of the rows would be misclassified aka not labeled right. 
 - 9738 would be labeled incorrectly
 - 70044 are the total amount of rows in the data set used in the model
 - 0.13903 is the error rate

Table 

1. CP (Complexity Parameter) is a value used to control the size of the decision tree and prevent overfitting. It acts as a penalty for adding more splits to the tree. A higher CP value makes the tree more conservative (fewer splits), helping to avoid overfitting by forcing the model to generalize more. A lower CP value allows more splits, which can lead to a very detailed tree that may fit the training data too closely — potentially capturing noise or outliers rather than true patterns.
Overfitting happens when a model fits the training data too perfectly — including rare exceptions or outliers — which hurts its ability to generalize to new data. In real-world applications, we aim for a model that captures the general structure of the data while ignoring noise.

2. rel error represents the model's error on the training data, relative to the root node error. For example, a value of 0.37061 means the model has reduced the error to 37.1% of the original baseline error (before any splits).

3. xerror is the cross-validated error rate — an estimate of how well the model would perform on unseen data. It’s the key metric for identifying potential overfitting.
4.  xstd is the standard deviation of the xerror value, giving you a sense of variability in the cross-validation results. It's often used with the 1-SE rule to choose a simpler, more robust model.

Training Data Results from Model 
```
predTrn = predict(lcDT1a, lcdfTrn, type = 'class')

confusionMatrix(predTrn, lcdfTrn$loan_status)

# results
Confusion Matrix and Statistics

             Reference
Prediction    Fully Paid Charged Off
  Fully Paid       58557        1522
  Charged Off       1663        8167
                                         
               Accuracy : 0.9544         
                 95% CI : (0.9529, 0.956)
    No Information Rate : 0.8614         
    P-Value [Acc > NIR] : < 2e-16        
                                         
                  Kappa : 0.8104         
                                         
 Mcnemar's Test P-Value : 0.01311        
                                         
            Sensitivity : 0.9724         
            Specificity : 0.8429         
         Pos Pred Value : 0.9747         
         Neg Pred Value : 0.8308         
             Prevalence : 0.8614         
         Detection Rate : 0.8376         
   Detection Prevalence : 0.8594         
      Balanced Accuracy : 0.9076         
                                         
       'Positive' Class : Fully Paid  

```

As a whole this model did very well on being able to distinguish  between Fully Paid and Charged Off loans compared to the actual results of the training data completing 95.4% accuracy! Now let's compare it to the testing data 


Testing Data Results from Model 

```
predtest = predict(lcDT1a, lcdfTest, type = 'class')

confusionMatrix(predtest, lcdfTest$loan_status)

Confusion Matrix and Statistics

             Reference
Prediction    Fully Paid Charged Off
  Fully Paid       25274         634
  Charged Off        687        3455
                                          
               Accuracy : 0.956           
                 95% CI : (0.9537, 0.9583)
    No Information Rate : 0.8639          
    P-Value [Acc > NIR] : <2e-16          
                                          
                  Kappa : 0.814           
                                          
 Mcnemar's Test P-Value : 0.1525          
                                          
            Sensitivity : 0.9735          
            Specificity : 0.8449          
         Pos Pred Value : 0.9755          
         Neg Pred Value : 0.8341          
             Prevalence : 0.8639          
         Detection Rate : 0.8411          
   Detection Prevalence : 0.8622          
      Balanced Accuracy : 0.9092          
                                          
       'Positive' Class : Fully Paid     


```

As a note, the reason we are using the type "class" code is to ensure when the function is running the testing set through the model, it is returning either " Fully Paid Off" or "Charged Off", if this code was not here the default would be a duo table with each row having probabilities for either option. 

The model achieved an accuracy of 95.6% on the test set, with high sensitivity (97.4%) and specificity 
(84.5%), indicating strong performance in correctly identifying both Fully Paid and Charged Off loans. The Kappa score of 0.81 suggests excellent agreement beyond chance, and McNemar’s test (p = 0.15) shows no significant bias between false positives and false negatives. Overall, the model generalizes well and maintains a balanced classification performance.

### ROC Curve 
![image](https://github.com/user-attachments/assets/1a7d3185-c635-442d-8942-53a03563ac2e)

- ROC Curve: This curve illustrates how well the model distinguishes between classes by plotting the true positive rate (sensitivity) against the false positive rate across different classification thresholds. A strong model will have a ROC curve that hugs the top-left corner of the graph, indicating high sensitivity and low false positives — which increases the Area Under the Curve (AUC).
An AUC of 0.5 represents random guessing (no discriminative ability), while an AUC of 0.954 for this model shows it can distinguish between "Fully Paid" and "Charged Off" loans with high effectiveness. This means the model can correctly rank borrowers by risk with 95.4% confidence, independent of a specific threshold.

- The red line is the baseline measurement, aka is this model better at distinguishing between classes
  better than guessing

### Lift Curve 

```
score=predict(lcDT1a,lcdfTest, type="prob")[,"Charged Off"]
pred = prediction(score, lcdfTest$loan_status, label.ordering = c("Fully Paid", "Charged Off"))

liftPerf <-performance(pred, "lift", "rpp")
plot(liftPerf, main = 'Lift Curve for Loan Status Model')


```

![image](https://github.com/user-attachments/assets/e8d1b015-df70-4885-8511-05850f70fda7)

I mentioned before that when I used the predict function, we wanted it to use the classication method signified as type = "class", the reason being is that usually the function returns probabilities if the target variable is x or y. Example can be seen below!

```
    Fully Paid Charged Off
1     0.99778873 0.002211271
2     0.99778873 0.002211271
3     0.95326520 0.046734796
4     0.99778873 0.002211271
6     0.99778873 0.002211271
7     0.95988935 0.040110650
8     0.99778873 0.002211271
9     0.95515371 0.044846293

```

The lift curve shows that, by setting "Charged Off" as the positive class, the model identifies high-risk loans with strong early precision. In the top 10% of predictions, it flags Charged Off loans at 7 times the rate of random guessing, indicating strong lift. This makes the model valuable for proactively identifying and denying high-risk loan applications, helping reduce potential losses.
