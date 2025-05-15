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

Insights:
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

This plot shows the range (max–min) of interest rates for each loan grade. We observe less variability in the early grades (A and B) as well as in the highest-risk grades (F and G). This suggests that applicants in these categories have more consistent profiles, and lenders are more confident in the expected outcome.
For example, Grades F and G exhibit consistently high interest rates with little deviation. This could indicate that lenders anticipate a higher risk of default and are less flexible on rate negotiation—they likely have a clear expectation of the return required to offset the risk.
Similarly, Grades A and B show lower and tighter interest rate distributions, likely reflecting high creditworthiness and less uncertainty, leading to more standardized loan terms.
