#  P2P Lending Risk & Return Analysis

This project explores the evolution and mechanics of peer-to-peer (P2P) lending platforms like LendingClub, Prosper, and Kiva. These platforms leverage fintech and data analytics to connect borrowers and investors, offering a low-cost alternative to traditional banking. With fintechs accounting for 38% of U.S. personal loans by 2018, the online lending industry has rapidly evolved. A pivotal milestone was LendingClub’s acquisition of Radius Bank in 2021, transitioning from a P2P platform to a full-spectrum digital bank.

###  Project Goals

1. **Predictive Modeling**: Evaluate multiple machine learning models in R to predict whether a loan will be *Fully Paid* or *Charged Off*.
2. **Investor Profitability**: Analyze which loan grades are most profitable for investors, comparing model predictions to actual financial returns.

---

##  Repository Structure

This repository is organized into modular folders to showcase different aspects of the analysis:

| Folder | Description |
|--------|-------------|
| `exploratory_analysis/` | Data cleaning, manipulation, and initial exploration of loan performance, grade breakdowns, and missing values |
| `random_forest/` | Random Forest model implementation and evaluation |
| `decision_tree/` | Decision Tree model and interpretation |
| `linear_model/` | Logistic Regression and linear classification modeling |
| `model_comparison/` | Side-by-side comparison of model performance metrics (e.g., accuracy, AUC) |
| `profitability_analysis/` | Analysis of predicted loan outcomes vs. actual returns to evaluate investor profitability by grade |

Each folder includes an R script or RMarkdown file with documentation and visuals.

---

##  Dataset

The analysis uses a cleaned subset of LendingClub data (`lcdfSample.csv`) containing loan-level features such as:
- Loan amount, interest rate, term
- Grade / sub-grade
- Employment length
- Annual income
- Loan status (target variable)



