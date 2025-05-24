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

  
