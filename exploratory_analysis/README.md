# Exploratory Analysis 

We want to create models to better use historical data to better understand if a peson will default on their loan or will pay off their loan. But we also want to know which types of loans are the most profitable. These are important questions to ask if you are a lender to ensure you are not losing money and to avoid as much risk as possible. But before we getting into modeling we have to ensure our data is cleaned and explore to see if we are able to see any relationships before creating mdodels.

In this section, I take a closer look at the data attributes and categorize them based on what they represent. Before conducting any analyses, I consider which attributes might be most important for the decision task at hand. I start off by looking at the data structure and the missing count for variables to reduce features. We'll come back to this. 

str(lcdf) allows us to see the data type of each feature as well as the number of rows as well as the number of variables. 
  - 99959 obs
  - 146 variables

We want to find the "NA" count per column next to understand the level of data manipulation we will need to do in either deletion or replacement. In order to do this we will need associate a number of missing values within each column and let's just sort from most NA's to least to get an idea of which variables are not important. 

missing_count = sort(colSums(is.na(lcdf)), decreasing = TRUE) 
  - The variable is called missing_count. is.na(lcdf) is creating a matrix for each entry and if there is an NA value it returns a true otherwise it will be False. ColSum() is adding the true values which equal 1 to give us a value for each column. Decreasing = True, is just returning the list from greatest sum count to least sum count. 
  - ![image](https://github.com/user-attachments/assets/38428eb9-7082-4972-8821-03dee8bdbe55)

Since we know how many NA values are in each column, we want to see how many columns have more than 50% of their values missing, if almost half of their values are missing they might not be worth inserting replacement data as there is not enough information that will give us variety across the NA values we are replacing.

total_row = nrow(lcdf) 
missing_cols = names(missing_count[missing_count > (0.5 * total_row)])
  - Here we find that 56 variables NA summation is over the 0.5 threshold. Below is an image of the output
  - For this line of code we are getting the names of variables or columns that are above the 50% threshold, if the column is over it will be assigned a value of TRUE. We will extract the names and store them in missing_cols.
  - ![image](https://github.com/user-attachments/assets/8303743e-2dfd-4b97-9442-4fd70fe17157)

We'll come back to this before going into the model portion to clean this up but this is good to know for now!


### Next, I calculate the proportion of defaults by comparing 'Charged Off' vs. 'Fully Paid' loans in the dataset. I then examine how default rates vary across loan grades and sub-grades to assess whether these features provide meaningful insights into the riskiness of different loans.

Loan Ststus Distribution
  - ![image](https://github.com/user-attachments/assets/5093d47e-b155-48b2-9b02-0150e363f13f)

Loan Status Grouped by Grade 
  - ![image](https://github.com/user-attachments/assets/48815585-50a3-4c98-822b-1d740c63e78b)

Loan Status Grouped by SubGrade 
  - ![image](https://github.com/user-attachments/assets/7af0314e-6f93-47ce-93f4-227e62b0b02b)

**Insights**:
An initial observation is the distribution between “Charged Off” and “Fully Paid” loans— loans are approximately 6.26 times more likely to be fully paid than charged off. This class imbalance is important to keep in mind during modeling, as it may skew predictions toward the majority class if not properly addressed. Additionally, when examining the combinations of loan status and grade, we see that Fully Paid loans in Grades B and C are the most common, followed by Grade A. Notably, Grade C also has the highest number of Charged Off loans, which may signal a higher risk associated with this grade, or it can be the most uncertain grade class given C ranks high in the Fully Paid off section as well. 


### Next, I analyze how many loans exist within each grade and examine whether loan amounts vary by grade. I also explore how these patterns differ based on loan status (e.g., Fully Paid vs. Charged Off). To support this analysis, I generate visualizations and summarize key takeaways based on the observed trends.

  - ![image](https://github.com/user-attachments/assets/bd875f20-e9a5-4294-910b-ac29a1dde4f1)
  - Loan Grade B has the highest total loan amount distributed across all loans. According to the data dictionary, the loan_amnt variable represents "the listed amount of the loan applied for by the borrower. If at some point in time, the credit department reduces the loan amount, then it will be reflected in this value."
  - To analyze this, we created a new variable total_loan, which is the sum of all loan amounts in the dataset.
  - We then grouped the data by grade, summarized the total loan amount per grade, and calculated each grade’s proportion of the total loan amount. This gives us a clear view of how loan distribution varies across different credit grades.

 ### Does interest rate for loans vary by grade, subgrade? Let's look at the average, standard- deviation, min and max of interest rate by grade and subgrade. 

   - ![image](https://github.com/user-attachments/assets/c51d0d94-7f81-4e8e-b407-f0626b5f32fc)
  - Let's note some key insights:
      - Loan grade B has the highest std meaning it can skew most to either being higher or lower than it's average
      - There is a inverse relationship between loan grades and average interest rates, the lower the grade the higher the interest rate average is, which makes sense and what would we would expect.
      - ![image](https://github.com/user-attachments/assets/e69eba78-a421-4c99-b8d1-64782aee6c77)

**Insights**: 
This plot shows the range (max–min) of interest rates for each loan grade. We observe less variability in the early grades (A and B) as well as in the highest-risk grades (F and G). This suggests that applicants in these categories have more consistent profiles, and lenders are more confident in the expected outcome.
For example, Grades F and G exhibit consistently high interest rates with little deviation. This could indicate that lenders anticipate a higher risk of default and are less flexible on rate negotiation—they likely have a clear expectation of the return required to offset the risk.
Similarly, Grades A and B show lower and tighter interest rate distributions, likely reflecting high creditworthiness and less uncertainty, leading to more standardized loan terms.

### How does interest rate relate with loan status and does this vary by grade?

![image](https://github.com/user-attachments/assets/b1854a4a-68af-4e00-869a-d1473a01605c)
  - Interest rate distribution by loan status


![image](https://github.com/user-attachments/assets/1ca445a2-7de1-4c67-b52e-8b3d37552c39)
  - Interest rate distribution by loan grade


**Insights**:

The boxplot comparison between interest rates and loan status reveals that while the overall spread (IQR) of interest rates is similar for both Fully Paid and Charged Off loans, the median interest rate is noticeably lower for loans that were fully repaid. This suggests that lower-risk borrowers tend to receive more favorable interest rates. Interestingly, there are several outliers—borrowers with very high interest rates—who still managed to fully repay their loans. In fact, some of the highest interest rates appear within the Fully Paid group. Since interest rates are determined before loans are issued, this pattern implies that repayment outcomes are influenced by more than just interest rate. Factors like income level, employment stability, or financial discipline likely contribute to whether a borrower successfully repays, even under costly borrowing terms.

### We want to see how long it actually takes for the loans that were actually paid back! We are going to call this new variable within the dataset "actual term" which is different from the "loan term".

**Proccess** 
  - In order to create this new variable we need to find the difference in time from the two variables labeled as last_payment_date and issue_d. The names are self explanatory, but by finding the time between when a loan was issued and the last payment date we can compare the actual term of the loan vs the loan term.
  - ![image](https://github.com/user-attachments/assets/7af9dc8e-389c-4c34-9bf3-6f47a881f866)
      - Here we can see the format is lining up so we would not be able to find the time between so we need to change the format for the last_payment_d variable and add a day (-01) as well as change the name of the month to a number
    - lcdf1$last_pymnt_d = paste(lcdf1$last_pymnt_d, "-01", sep = "") this will create the day portion by adding to the string.
    - lcdf1$last_pymnt_d = parse_date_time(lcdf1$last_pymnt_d, "myd") this will change the format to the format issue_d is in where the year is first, month, and day at the end.
    - Now we can create the new variable called "actualTerm".
        - lcdf1$actualTerm = ifelse(lcdf$loan_status=="Fully Paid", as.duration(lcdf1$issue_d  %--%       lcdf1$last_pymnt_d)/dyears(1), 3)
            - If the loan is fully paid, calculate the number of years between the issue date and last payment date. If the loan is not fully paid, assume it had a 3-year term.
    - ![image](https://github.com/user-attachments/assets/7ee1b2f1-212a-41b9-a04b-4a57ad3688d8)
        - Now we can how long each loan took to repay the loan vs the term of the loan.
  - Finally we can see a distribution of the actual term length for paid off loans
      - ![image](https://github.com/user-attachments/assets/34cd5713-8a54-44cc-910a-813b058bcbe1)
  - We can also see the distribution by grade
      - ![image](https://github.com/user-attachments/assets/51646cfa-603b-402b-88de-1ebce605a391)

**Insights**: 

Most fully paid loans are repaid in under 2.5 years, which is shorter than the expected 36-month term—indicating early repayment is common. When broken down by grade, we observe that lower-grade loans tend to have slightly longer repayment times, potentially reflecting more financial strain or slower payoff behavior. This is important for investors weighing the tradeoff between higher interest rates and longer capital commitment.


#### We are going to create the annual return from each loan. We are going to use an ifelse statement checking if the actual term is greater than 0, if it is, we are going to subject the total amount that was paid on the loan vs the amount funded on the loan. We'll want this to be a percentage. So once we have the profit on the loan we'll divide it by the origianl amount to get a %. Since there are varying degrees of when a loan is paid off as we saw in the last boxplot, we'll want to annualize this percentage because right now this is a percentage that represents percent profit over the entirety of the loan. So we need to divide 1/actual term of the loan. So let's say we paid 10000 on a 8000 loan. That extra 2000 was because of interest. We paid off the loan in 2 years. Let's plug in the numbers. 

    - 10,000 - 8,000 = 2,000 profit from this loan 
    - 2,000/ 8,000 = 0.25 return on the original investment 
    - 0.25 * 1 year/ 2 years it took me to pay back the loan = 0.125 
    - Let's mutiply this number by 100 to get a percentage 0.125 * 100 = 12.5% 
    - So year over year this loan would generate us 12.5% return on investment. 
    - Code: lcdf1$actualReturn <- ifelse(lcdf1$actualTerm>0, ((lcdf1$total_pymnt -       lcdf1$funded_amnt)/lcdf1$funded_amnt)*(1/lcdf1$actualTerm)*100, 0)

- ![image](https://github.com/user-attachments/assets/fb39bf12-b857-424e-b184-b297bae3594a)
    - We can see some examples of this being applied to our current dataset to get an idea of which loans yield the greatest year over year return on investments.

 #### Next we are going to curate a new variable that is similar to the last actual return variable we created but this time we are assuming that the team of each loan is 36 months rather than the actual term. The reason I am doing this is because in real world models there might be fixed times to get an idea of ROI's based on fixed time periods, here we can see the difference between a set fixed period and how it's annual returns compare to actual returns derived from actual pay back time. 

  - Code: lcdf1$annRet <- ((lcdf1$total_pymnt -lcdf1$funded_amnt)/lcdf1$funded_amnt)*(12/36)*100
  - ![image](https://github.com/user-attachments/assets/77c3d299-5d71-4b63-8838-433913c685ee)
  - 
        - Snapshot of the same table shown above, but now with annual returns under the three year fixed term assumption and we can see there is an underestimation of returns. 

 
#### In this section, I examine whether investors lose their entire investment when loans are charged off. I analyze how returns on charged-off loans vary across loan grades and compare the average return values to the average interest rates to identify any meaningful differences. Additionally, I evaluate how returns vary by both grade and sub-grade to better understand risk-reward dynamics across loan categories. Based on these insights, I reflect on which types of loans may offer the most attractive investment opportunities.
 

![image](https://github.com/user-attachments/assets/d16262cb-7d11-4dc1-84c7-3abc06a0b4a0)

- We want to first create a new data set here with only the loans that are "charged off" as we want to see how much money was lost here, so we'll also create a new column called difference where we are subtracting the funded amount of the loan and the amount paid on the loan.
- After, I want to create a summarization table using this new dataset to get aggregate data across the grade loans, here I can do this easily now by using different aggregatie functions on the "difference" variable that was created.
    - ![image](https://github.com/user-attachments/assets/8a032112-e659-4b35-903c-73e41116bd0a)

**Insights**: 

1. Grade G loans exhibit the highest interest rates but result in the lowest actual returns, suggesting a poor risk-reward tradeoff for investors.
2. Grade G also accounts for the highest total loss and the largest number of loans that were charged off, further emphasizing the elevated risk associated with these loans.
3. Grade A loans show the highest average dollar loss per charged-off loan, but also have the highest average funded amounts. This suggests that lenders had strong confidence in Grade A borrowers, likely due to their low default risk, which justified issuing larger loan amounts. 


### Data Cleaning Before Modeling 

Before we run this dataset into any model to determine if we can predict if a loan will either be considered "Fully Paid" or "Charged Off", let's clean this data first to ensure we are removing variables that sre not important to the model. 

The first thing I am going to do before doing any  significant imputing for NA values, is delete variabls where all the values are NA. 
    


```{r}
# this is checking with columns have only missing values, these will be removed. 
lcdf1 %>% select_if( function(x) { all(is.na(x)) } ) %>% colnames() 

#Drop vars with all empty values # after we do this we are down to 112 variables.
lcdf1 <- lcdf1 %>% select_if(function(x){ ! all(is.na(x)) } )

# let's see the proportions of missing values within a dataset 

# here we are creating a empty data frame for now. the variable will be the column names, the missing ratio will be the proportion of the values within 
missing_info = data.frame( 
    variable = character(),
    missing_ratio = numeric(),
    stringsAsFactors = FALSE
)

for (col_name in names(lcdf1)) {
    na_count = sum(is.na(lcdf1[[col_name]])) # this is counting the number of NA within the columns and returning a number 
    na_ratio = round(na_count/nrow(lcdf1),2) # that number of NA is being divided by the number of rows within the entire data set to give us a ratio, and we want to round the result to the second decimal.
    missing_info = rbind(
        missing_info,
        data.frame( variable = col_name, missing_ratio = na_ratio, stringsAsFactors = FALSE)
    ) # here we are simply adding the varibales we just created into that empty data frame called missing_info
}
```

Here is a snippet of the missing_info dataframe we were able to create using the code above
  - ![image](https://github.com/user-attachments/assets/2006193f-a841-4444-a653-f3ec772f2389)

Now that we have the ratios of the values within each variable, we are able to create some sort of threshold where we able to delete variables not because all their values are NA but if a a certain percentage of their values are NA values, in this case we'll apply a 50% threshold. So if a variable has over 50% of their values being an NA value we'll delete them from thsi dataset called "lcdf2"

```{r}
# this shows the names of the variables with missing rations higher than 0.5, we are removing them
cols_to_remove <- missing_info$variable[missing_info$missing_ratio > 0.5] 

# we are removing the names extracted from the cols_to_remove, if the name appears in both datasets 
# the column will be removed.
lcdf2 <- lcdf1[, !(names(lcdf1) %in% cols_to_remove)]

```

After deleting the variables that are exceeding this threshold we are left with only 92 variables! 

Now with the variables that are left, let's make sure they are not left with NA values because although we know they are not all NA values or more than 50% NA values, they can be like 49% but we can work with it for now, so let's replace the values depending on the data type.

```{r}
fill_missing_values <- function(df) { # creating a function that is called fill_missing_values
  for (col_name in names(df)) { 
    if (anyNA(df[[col_name]])) { # is the column has any NA values we proceed 

      # Check the column type
      if (is.numeric(df[[col_name]])) {
        # 1. For numeric columns: fill with the median
        median_val <- median(df[[col_name]], na.rm = TRUE) # calculating the median value to replace the NA values, this median value is not taking into account the NAs. 
        df[[col_name]][is.na(df[[col_name]])] <- median_val # find all the NA values and replace with median_value 

      } else if (is.character(df[[col_name]])) { # checking to see if the column is a character
        # 2. For character columns: fill with "missing"
        df[[col_name]][is.na(df[[col_name]])] <- "missing" # if it is, than replace the NA value with "missing"

      } else if (is.logical(df[[col_name]])) {
        # 3. For logical columns: fill with the majority value (TRUE or FALSE)
        count_true <- sum(df[[col_name]] == TRUE, na.rm = TRUE)
        count_false <- sum(df[[col_name]] == FALSE, na.rm = TRUE)

        if (count_true >= count_false) {
          df[[col_name]][is.na(df[[col_name]])] <- TRUE
        } else {
          df[[col_name]][is.na(df[[col_name]])] <- FALSE
        }
      }
    }
  }
  return(df) # 
}


lcdf2 <- fill_missing_values(lcdf2) # is returning the new dataset into a new dataset  that is replacing the old dataset
lcdf2 = na.omit(lcdf2) # anything that is leftover from the function just omit the row

# after running the dataf rame lcdf2 through the if else statement and the na.omit function, I realized we did not add a statement for dates, so here I saw there were 64 NA values within the last_pymnt_d column so I am going to remove them. 
lcdf2 <- lcdf2[!is.na(lcdf2$last_pymnt_d), ]

missing_count = sort(colSums(is.na(lcdf2)), decreasing = TRUE) # this is a vector list 

# I am using the same code as the in the exploration portion to find out if there are missing values within the columns, there are not 
missing_df = data.frame( Column_Name = names(missing_count), Missing_Values = as.vector(missing_count))

```

In the previous step, we focused on handling missing values for numeric, logical, and character variables. For numeric columns, we replaced NA values with the median of that column. For logical variables—which contain only TRUE or FALSE values—we counted the frequency of each and filled missing values with the majority value. For example, if a column had 100 TRUEs and 50 FALSEs, we assigned all NAs in that column to TRUE.
After handling those specific data types, we applied na.omit() to the dataset as a final cleanup step. This ensures that any remaining missing values—possibly from data types we didn't explicitly cover—are removed.
When checking for missing values again using missing_count, I noticed that the last_pymnt_d column (a date column) still had 64 NAs. Since our custom function didn’t account for date types, I wrote a line of code to specifically remove rows where last_pymnt_d was missing. The logic used was:
```
lcdf2 <- lcdf2[!is.na(lcdf2$last_pymnt_d), ]

```

This line keeps only the rows where last_pymnt_d is not missing (!is.na(...)), ensuring the dataset is fully cleaned before moving on to modeling.

```
sort(missing_df, decreasing = TRUE)

```

![image](https://github.com/user-attachments/assets/6ed4ddac-8dd6-4dcf-844f-5e437c1515ce)


We're all good, there are no more NA values, let's start modeling!


