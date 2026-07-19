# HR Analytics: 2M Rows Raw & Cleaned Dataset

## 1. Project Overview
This dataset provides a large-scale simulation of Human Resources data, designed for **SQL benchmarking, Python data cleaning, and Power BI visualization**. It contains **2,000,000 records** of synthetic employee data, featuring intentional real-world "dirty data" challenges.

---

## 2. The Challenge (Raw Data)
The `Raw` version of this dataset is intentionally "broken" to test a Data Analyst's logic. It contains over **48,000+ logical inconsistencies**, including:
* **Age-Experience Gaps:** Employees younger than their years of experience + education.
* **Invalid Salaries:** Negative or zero salary values.
* **Hierarchical Errors:** Junior employees with higher salaries than Directors.
* **Missing Values:** Strategic `NaNs` in performance ratings and departments.

---

## 3. The Solution (Cleaned Data)
The `Cleaned` version (available in the ZIP) has been fully restored using advanced Python techniques:
* **Logical Alignment:** Adjusted Age/Experience based on a strictly enforced $Age = Exp + 22$ baseline.
* **Stratified Imputation:** Filled missing `Performance_Rating` using **Stratified Random Sampling** to maintain departmental distribution.
* **Multi-level Median Filling:** Fixed salaries by calculating medians based on **Job Level + Department** groups.
* **Type Casting:** Optimized memory by converting Data Types (e.g., Float to Int).

---

## 4. Dataset Structure
| Column | Description |
| :--- | :--- |
| `Employee_ID` | Unique identifier for each staff member. |
| `Age` | Employee age (Validated in Cleaned version). |
| `Department` | Business units (Sales, IT, HR, Finance, etc.). |
| `Job_Level` | Hierarchy: Junior, Mid, Senior, Director. |
| `Years_at_Company` | Total tenure at the current organization. |
| `Salary` | Monthly/Annual compensation (Fixed for negatives). |
| `Performance_Rating` | Descriptive rating (e.g., Good, Satisfactory, Needs Improvement). |
---

## 5. Recommended Use Cases
* **SQL:** Practice complex `JOINs`, `Window Functions`, and `CTEs` on a large dataset.
* **Power BI / Tableau:** Build HR Dashboards (Attrition, Salary Distribution, Tenure).
* **Python:** Practice `Pandas` profiling and `NumPy` vectorization for Big Data.

---

## 6. How to Use
1. Download the `hr_dataset.zip`.
2. Use the `hr_raw.csv` CSV if you want to practice **Data Cleaning**.
3. Use the `hr_clean.csv` CSV for immediate **Visualization or Machine Learning**.