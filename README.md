# Retail Data Pipeline Project: ETL Solution Using Python

## Project Overview

This project aims to build an ETL pipeline for analyzing the supply and demand patterns around the holiday season. The pipeline involves 4 key stages:
1. Extraction
2. Transformation
3. Load
4. Validation

## Data Sources

- grocery_sales table: Contains key sales data [check out columns in the table](https://github.com/Choiceugwuede/Retail_Data_Pipeline/blob/main/notebook.ipynb)
- extra_data.parquet file: Contains additional data to be merged with the main dataset. [check out details here](https://github.com/Choiceugwuede/Retail_Data_Pipeline/blob/main/extra_data.parquet)

## Pipeline Steps

### Extraction Layer

This step merges the grocery_sales table with the extra_data.parquet file.
```Python
import pandas as pd
import os

# Extract function is already implemented for you 
def extract(store_data, extra_data):
    extra_df = pd.read_parquet(extra_data)
    merged_df = store_data.merge(extra_df, on = "index")
    return merged_df

# Call the extract() function and store it as the "merged_df" variable
merged_df = extract(grocery_sales, "extra_data.parquet")
```

### Transformation Layer

This step handles missing values, formats dates, filters the data, and selects relevant rows and columns for further analysis.
```Python
def transform(raw_data):
    #fill in missing values with median
    raw_data.fillna(raw_data.median(), inplace=True)
    #drop rows where date is missing
    raw_data = raw_data.dropna(subset=['Date'])
    #convert Date to datetime and making sure all missing values is replaced by default value
    raw_data['Date'] = pd.to_datetime(raw_data['Date'], errors='coerce')
    #drop exra rows that couldntt be converted
    raw_data = raw_data.dropna(subset=['Date'])
    #creating new column month
    raw_data['Month'] = raw_data['Date'].dt.month
    #keeping only rows with sales more than 10000
    raw_data = raw_data[raw_data['Weekly_Sales'] > 10000]
    #Filtering dataframe to only needed columns
    needed_column=['Store_ID','Month','Dept','IsHoliday','Weekly_Sales','CPI','Unemployment']
    raw_data=raw_data[needed_column]
    return raw_data
```

### Analysis Table

This step calculates the average weekly sales for each month.
```Python
def avg_weekly_sales_per_month(clean_data):
    result = (
        clean_data[['Month','Weekly_Sales']]
        .groupby('Month')
        .agg({'Weekly_Sales':'mean'})
        .reset_index()
        .round(2)
        .rename(columns={'Weekly_Sales': 'Avg_Sales'})
        
    )
    
    return result
agg_data=avg_weekly_sales_per_month(clean_data)
```

### Loading layer

This step saves the final cleaned and aggregated datasets to CSV files.
```Python
def load(full_data, full_data_file_path, agg_data, agg_data_file_path):
    full_data.to_csv(full_data_file_path,index=False)
    agg_data.to_csv(agg_data_file_path,index=False)
    pass
```

### Validation Layer

This step checks if the output file exists after the loading step.
```Python
def validation(file_path):
    data_exists = os.path.isfile(file_path)
    
    # Return True if file exist, otherwise False
    if data_exists:
        return True
    else:
        return False
    pass
```

### Conclusion

This pipeline enables efficient analysis of sales trends during the holidays by processing large data sets in a structured and validated manner

 
