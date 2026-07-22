# Data Science Portfolio

A collection of end-to-end data analysis projects — each folder is a self-contained study with its cleaning, EDA, modelling, and a final report. Raw datasets are **not** committed (they're large; download links live in each project's README); the notebooks below render on GitHub with all charts and outputs baked in.

## Projects

| Project | Notebooks | Report |
|---|---|---|
| **Avocado Prices** | [EDA & cleaning](Avocado%20Prices/notebooks/01_get_know_clean_eda.ipynb) · [Deep dive](Avocado%20Prices/notebooks/02_deep_dive_paths_abc.ipynb) | [final_report.html](Avocado%20Prices/reports/final_report.html) |
| **Adidas Global Catalogue 2026** | [Ingestion](Adidas%20Global%20Catalogue%202026/notebooks/01_ingestion.ipynb) · [Cleaning](Adidas%20Global%20Catalogue%202026/notebooks/02_cleaning.ipynb) · [EDA](Adidas%20Global%20Catalogue%202026/notebooks/03_eda.ipynb) · [Analysis](Adidas%20Global%20Catalogue%202026/notebooks/04_analysis.ipynb) | [final_report.html](Adidas%20Global%20Catalogue%202026/reports/final_report.html) |
| **Olist Brazilian E-Commerce** (tabular + PT-BR text) | [Ingestion](Brazil%20E-commerce/notebooks/01_ingestion.ipynb) · [Cleaning](Brazil%20E-commerce/notebooks/02_cleaning.ipynb) · [EDA](Brazil%20E-commerce/notebooks/03_eda.ipynb) · [Analysis](Brazil%20E-commerce/notebooks/04_analysis.ipynb) · [Text](Brazil%20E-commerce/notebooks/05_text_analysis.ipynb) · [Reporting](Brazil%20E-commerce/notebooks/06_reporting.ipynb) | [final_report.html](Brazil%20E-commerce/reports/final_report.html) |
| **Boston House Prices** | [Ingestion](Boston%20House%20Prices/notebooks/01_ingestion.ipynb) · [Cleaning](Boston%20House%20Prices/notebooks/02_cleaning.ipynb) · [EDA](Boston%20House%20Prices/notebooks/03_eda.ipynb) · [Analysis](Boston%20House%20Prices/notebooks/04_analysis.ipynb) · [Reporting](Boston%20House%20Prices/notebooks/05_reporting.ipynb) | [final_report.html](Boston%20House%20Prices/reports/final_report.html) |
| **Toy Store E-Commerce** | [Ingestion](Toy%20Store%20E-Commerce%20Database/notebooks/01_ingestion.ipynb) · [Cleaning](Toy%20Store%20E-Commerce%20Database/notebooks/02_cleaning.ipynb) · [EDA](Toy%20Store%20E-Commerce%20Database/notebooks/03_eda.ipynb) · [Analysis](Toy%20Store%20E-Commerce%20Database/notebooks/04_analysis.ipynb) · [Reporting](Toy%20Store%20E-Commerce%20Database/notebooks/05_reporting.ipynb) | [final_report.html](Toy%20Store%20E-Commerce%20Database/reports/final_report.html) |
| **HR Analytics (2M rows)** | [Ingestion](HR%20Dataset%20Clean%20%26%20Raw/notebooks/01_ingestion.ipynb) · [Cleaning](HR%20Dataset%20Clean%20%26%20Raw/notebooks/02_cleaning.ipynb) · [EDA](HR%20Dataset%20Clean%20%26%20Raw/notebooks/03_eda.ipynb) · [Analysis](HR%20Dataset%20Clean%20%26%20Raw/notebooks/04_analysis.ipynb) · [Reporting](HR%20Dataset%20Clean%20%26%20Raw/notebooks/05_reporting.ipynb) | [final_report.html](HR%20Dataset%20Clean%20%26%20Raw/reports/final_report.html) |
| **Credit Card Fraud Detection** (284k txns, 0.17% positive) | [Ingestion](Credit%20Card%20Fraud%20Detection/notebooks/01_ingestion.ipynb) · [Cleaning](Credit%20Card%20Fraud%20Detection/notebooks/02_cleaning.ipynb) · [EDA](Credit%20Card%20Fraud%20Detection/notebooks/03_eda.ipynb) · [Diagnostic](Credit%20Card%20Fraud%20Detection/notebooks/04_diagnostic.ipynb) · [Modelling](Credit%20Card%20Fraud%20Detection/notebooks/05_modeling.ipynb) · [Reporting](Credit%20Card%20Fraud%20Detection/notebooks/06_reporting.ipynb) | [final_report.html](Credit%20Card%20Fraud%20Detection/reports/final_report.html) |
| **Superstore** | [superstore.ipynb](Superstore%20Dataset/superstore.ipynb) | — |
| **Telco Customer Churn** | [telco.ipynb](Telco%20Customer%20Churn/telco.ipynb) | — |
| **Fraud Detection (1M transactions)** | [notebook](Fraud%20detection%201M%20transactions/datasets/Fraud%20detection%201M%20transactions.ipynb) | — |

> **Note on HTML reports:** GitHub displays `.html` files as source rather than rendering them. To view a report in your browser, open it via [htmlpreview.github.io](https://htmlpreview.github.io/) or download and open it locally.

## Repository structure

Only notebooks (`.ipynb`), HTML reports, and READMEs are tracked. Data files, source scripts, and generated figures are excluded via `.gitignore` to keep the repo lightweight.
