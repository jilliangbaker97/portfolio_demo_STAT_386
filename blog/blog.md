# From Raw CSV to Clean Dataset in 10 Steps

## A Reproducible Data Cleaning Workflow in Python
Raw datasets are rarely analysis-ready. Column names may be inconsistent, data types incorrect, missing values unexplained, and duplicate rows quietly distorting results. At first glance, a CSV file may appear usable—but once explored more carefully, small issues quickly compound into larger problems. If these issues go unchecked, exploratory data analysis (EDA) can become misleading, and any models built on top of the data may produce unreliable conclusions.

In real-world settings, data rarely comes clean. Industry estimates often suggest that data scientists spend **60–80% of their time cleaning and preparing data** rather than building models. While modeling and visualization often receive the most attention, high-quality analysis depends on well-structured, trustworthy data.

This is where a **structured, reproducible workflow** becomes essential. Instead of cleaning data randomly or reactively, following a consistent step-by-step process improves clarity, reproducibility, and precision. A defined workflow helps prevent simple mistakes—such as overwriting raw files or misinterpreting data types—and creates a solid foundation for everything that follows. In this post, I’ll walk through a practical 10-step process for transforming a raw CSV file into a clean, analysis-ready dataset.

Common issues in raw datasets include:

- Inconsistent column names  
- Mixed or incorrect data types  
- Missing values without explanation  
- Duplicate rows  
- Formatting inconsistencies (extra whitespace, currency symbols, mixed date formats)  

### Before and After Cleaning

| Messy Dataset | Clean Dataset |
|---------------|--------------|
| ![Messy CSV showing inconsistent column names and formatting issues](/images/messy.png) | ![Clean CSV showing standardized column names and formatted values](/images/clean.png) |

The raw dataset contains inconsistent capitalization, mixed date formats, and currency symbols embedded within numeric fields. After applying the workflow, the cleaned dataset reflects standardized naming conventions, properly typed variables, and a structure ready for reliable analysis.

---

## The 10-Step Workflow

### Step 1: Import Libraries

```python
import pandas as pd
import numpy as np
```

Reproducible research, where data, code, and methods are documented so that the same analysis can be re-run later, is a foundational concept in scientific data work. It promotes transparency and ensures that results can be verified, audited, or extended by others. Just as importantly, reproducibility benefits your future self. When every transformation and decision is clearly recorded, you can retrace your steps months later without relying on memory or guesswork. This significantly reduces errors caused by undocumented changes or inconsistent processes.

Consistency plays a major role in making reproducibility practical. By following the same structured cleaning workflow for each dataset, you reduce the time spent deciding what to do next and increase the time spent understanding the data itself. Repeating a clear process builds familiarity and confidence, allowing you to identify issues more quickly. It also prevents scrambling when a dataset feels overwhelming or unfamiliar. Instead of reacting randomly to messy data, you move through a deliberate sequence of steps—turning chaos into a manageable, repeatable system.

### Step 2: Load the Data
```python
df = pd.read_csv("data/raw_data.csv")
```

Keeping **raw data untouched** is essential in any data workflow. If something goes wrong during cleaning or you later decide to approach the analysis differently, you need the ability to restart from the original source. Overwriting or modifying raw files removes that safety net.

It is especially risky to delete values simply because they seem unusual or unclear. For example, what appears to be an “unknown” or outlier value may actually contain meaningful information. Removing or altering it too early can limit your understanding of the dataset and potentially bias your analysis.
Preserving the raw data ensures that all original inputs remain available for review. This allows you to revisit assumptions, test alternative cleaning strategies, or reuse the dataset for a different project with new research questions. A best practice is to treat raw data as read-only and perform all cleaning and transformations on a separate copy. That way, your workflow remains both flexible and reproducible. Reproducibility is widely recognized as a core principle of scientific computing and data analysis (see [The Turing Way’s guide to reproducible research](https://the-turing-way.netlify.app/reproducible-research/reproducible-research.html)).

### Step 3: Inspect the Dataset
```python
df.head()
df.info()
df.describe()
```
Carefully inspecting a dataset prevents downstream errors by revealing its current structure and potential inconsistencies. Reviewing column names, data types, and summary statistics helps identify issues early, such as mislabeled variables or unexpected missing values. It also highlights whether the data needs to be reshaped—through melting, pivoting, or aggregation—before analysis can proceed. By thoroughly examining the dataset upfront, you reduce the risk of mistakes later in the workflow and ensure a more reliable and efficient cleaning process.


### Step 4: Standardize Column Names
```python
df.columns = df.columns.str.strip().str.lower().str.replace(" ", "_")
```
Clean naming conventions typically use lowercase letters with underscores in place of spaces (often called snake_case). This format improves readability and keeps column names consistent across datasets and projects. Spaces in column names can create subtle issues, especially when referencing variables in code. For example, column names with whitespace may require bracket notation instead of dot notation, which can interrupt workflow and increase the chance of small syntax errors.
Maintaining consistent naming conventions across all datasets supports reproducibility. When column names follow predictable patterns, you can reuse code more easily without constantly renaming variables or adjusting scripts. Over time, this consistency reduces friction in your workflow and allows you to focus on analysis rather than formatting issues. Clean, standardized naming may seem like a small detail, but it plays a significant role in creating efficient and reliable data processes.

### Step 5: Handle Missing Values
```python
df.isnull().sum()
df = df.dropna()
```
The [`dropna()` function](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.dropna.html) removes rows or columns that contain missing (NA) values. By default, it drops any row with at least one missing value, though this behavior can be adjusted with parameters. While this can simplify a dataset quickly, it should be used with caution. Automatically removing rows may discard meaningful observations, especially if the missingness is limited to a single variable that is not central to your analysis.
Before using dropna(), it is important to consider whether complete cases are truly required. In some analyses, every column must contain a value for the observation to be usable. In other situations, however, a row with one missing entry may still provide valuable information. Removing it could reduce sample size or introduce bias if the missing data follows a pattern.
A better practice is to first assess the extent and structure of missing values. From there, you can decide whether to drop observations, impute values, or leave them as-is depending on your analytical goals. Thoughtful handling of missing data leads to more reliable and transparent results.

### Step 6: Fix Data Types
```python
df["date"] = pd.to_datetime(df["date"])
df["price"] = df["price"].astype(float)
```
### Step 7: Remove Duplicates
```python
df = df.drop_duplicates()
```
### Step 8: Filter or Subset Data
```python
df = df[df["price"] > 0]
```
Applying logical constraints ensures that your dataset reflects realistic and meaningful values. Constraints involve filtering out observations that violate known rules of the problem—for example, negative prices, impossible dates, or ages greater than 120. These values may result from data entry errors, formatting issues, or incorrect merges. If left unchecked, they can distort summary statistics, mislead visualizations, and negatively impact model performance. By enforcing logical boundaries based on domain knowledge, you improve data quality and ensure that your analysis is grounded in plausible, interpretable observations rather than errors.

### Step 9: Create Derived Variables
```python
df["revenue"] = df["price"] * df["quantity"]
```
**Feature engineering** often begins during the cleaning stage because this is when raw variables are transformed into more meaningful and usable forms. As you standardize types, handle missing values, and apply logical constraints, you naturally identify opportunities to create new variables—such as calculating revenue from price and quantity, extracting month or year from a date column, or generating indicator variables from categories. These derived features can better capture underlying patterns in the data and improve model performance. Rather than viewing cleaning and feature engineering as separate steps, it is more accurate to see cleaning as the foundation where thoughtful variable construction first takes shape.

### Step 10: Save the Clean Dataset
```python
df.to_csv("data/clean_data.csv", index=False)
```
Saving the cleaned dataset as a new file is a crucial final step in maintaining a reproducible workflow. Instead of overwriting the original raw data, export the processed version to a separate location (for example, a processed or cleanfolder) using df.to_csv("data/clean_data.csv", index=False). Keeping raw and cleaned data separate preserves the integrity of the original source and ensures you can restart the process if needed. Separating stages of data—raw, intermediate, and cleaned—also makes your project easier to audit, debug, and extend, reinforcing a clear and organized analytical pipeline.


## Data cleaning is not just preparation

Data cleaning is the foundation of reliable analysis. By following this structured 10-step workflow, you transform messy CSV files into trustworthy datasets ready for exploration and modeling.

The next time you download a dataset, resist the urge to jump straight into visualization or modeling. Instead, walk through these ten steps deliberately and document each transformation. You’ll not only improve your results—you’ll build a reproducible system you can reuse across projects.

In a future post, I’ll apply this workflow to a real-world dataset and demonstrate how structured cleaning improves downstream analysis.

Reliable analysis begins with reliable data.
