# Analysis of Child Labour Worldwide

This project explores child labour across countries and regions, analysing trends over time and differences by gender, region, and socioeconomic factors. The workflow included data import, cleaning, analysis, and database storage.

1. Notebook Setup and Data Import

Import essential Python libraries for data handling and visualisation:

    import pandas as pd

    import numpy as np

    import matplotlib.pyplot as plt

    import seaborn as sns


The dataset is an Excel file containing multiple sheets. We used ExcelFile to inspect the sheets and read the relevant "Child labour" sheet:

    file = pd.ExcelFile(r'C:\Users\matti\OneDrive\Desktop\MASTER\Module 3\Unit 9\Seminar\unicef_oct_2014.xls')

    file.sheet_names  # Check sheet names

    df = pd.read_excel(file, sheet_name='Child labour  '

    df.head()

2. Data Cleaning and Preparation

The raw dataset contains non-informative rows and irregular headers. We kept only relevant rows and reset the indices:

    new_df = df.reset_index().loc[3:129, :]

    new_df = new_df.loc[2:].reset_index(drop=True)

    new_df = new_df.loc[1:109]  # Keep only useful rows

<img width="805" height="481" alt="image" src="https://github.com/user-attachments/assets/f4636243-142d-40a0-8bfe-0f56beeb520f" />

Column headers were renamed for clarity:

    new_df = new_df.rename(columns={

    new_df.columns[0]: 'Country',
    
    new_df.columns[1]: 'Total_pct',
    
    new_df.columns[2]: 'y',
    
    new_df.columns[3]: 'Sex_pct_Male',
    
    new_df.columns[4]: 'Sex_pct_Female',
    
    new_df.columns[5]: 'Place_of_residence_Urban',
    
    new_df.columns[6]: 'Placde_of_residence_Rural',
    
    new_df.columns[7]: 'Household_wealth_quintile_Poorest',
    
    new_df.columns[8]: 'Household_wealth_quintile_Second',
    
    new_df.columns[9]: 'Household_wealth_quintile_Middle',
    
    new_df.columns[10]: 'Household_wealth_quintile_Fourth',
    
    new_df.columns[11]: 'Household_wealth_quintile_Richest',
    
    new_df.columns[12]: 'Reference_Year',
    
    new_df.columns[13]: 'Data_Source'
    })

<img width="882" height="463" alt="image" src="https://github.com/user-attachments/assets/e4732090-2eed-4950-8208-930d5dddba1d" />

3. Adding Geographic Regions

To analyse geographic trends, a Region column was added. Countries were mapped to regions using a predefined dictionary:


    country_to_region = {

    "Albania": "Balkans", "Argentina": "South America", "Afghanistan": "Central Asia",
    
    "Bangladesh": "South Asia", "Algeria": "North Africa", "Angola": "East Africa",
    
    # ... include all other mappings
    }

    new_df['Region'] = new_df['Country'].map(country_to_region)

<img width="889" height="812" alt="image" src="https://github.com/user-attachments/assets/5af3a8b2-4df8-4ab5-94b8-8e2a8bc0ae6d" />

This enabled regional analyses, such as comparing child labour in Africa versus Europe or Asia.

4. Formatting Reference Years

The Reference_Year column had inconsistent formats (e.g., "2011/2012", "2007-2008"). We extracted the starting year for analysis:

    for index, val in new_df['Reference_Year'].items():

    val_str = str(val)
    
    if '-' in val_str:
    
        new_df.at[index, 'Reference_Year'] = val_str.split('-')[0]
        
    elif '/' in val_str:
    
        new_df.at[index, 'Reference_Year'] = val_str.split('/')[0]
        
    else:
    
        new_df.at[index, 'Reference_Year'] = val_str
        
<img width="883" height="688" alt="image" src="https://github.com/user-attachments/assets/f6924189-c8a3-4d83-a2e2-d9134143f322" />

5. Handling Missing Values and Numeric Conversion

The dataset contained placeholders like '-' for missing values. These were replaced with NaN, and the relevant columns were converted to numeric types:

    new_df = new_df.replace('-', np.nan)

    cols_to_numeric = [

    'Placde_of_residence_Rural', 'Household_wealth_quintile_Poorest',
    
    'Household_wealth_quintile_Second', 'Household_wealth_quintile_Middle',
    
    'Household_wealth_quintile_Fourth', 'Household_wealth_quintile_Richest'
    ]

    for col in cols_to_numeric:

    new_df[col] = pd.to_numeric(new_df[col], errors='coerce')

<img width="884" height="703" alt="image" src="https://github.com/user-attachments/assets/2f04bf18-ed1e-4839-8196-900e4cf50667" />

6. Exploratory Data Analysis

Global Trend Over Time

We examined child labour percentages from 2000 to 2013. Overall, rates were fairly stable, except for a notable spike in 2008 caused by countries like Bolivia and Nepal:

    new_df['Reference_Year'] = pd.to_datetime(new_df['Reference_Year'], format='%Y').dt.year

    new_df.groupby('Reference_Year')['Total_pct'].mean().sort_index().plot(kind='bar')

    plt.ylabel('Child Labour (%)')

    plt.show()

<img width="881" height="677" alt="image" src="https://github.com/user-attachments/assets/dad9bf11-8189-4b47-ad47-2a6035632311" />

Regional Analysis

African countries had the highest child labour rates, whereas Central European countries had the lowest:

    new_df.groupby('Region')['Total_pct'].mean().sort_values().plot(kind='bar')

    plt.show()

<img width="881" height="612" alt="image" src="https://github.com/user-attachments/assets/7e74ed03-86e7-411d-ad77-a8acc53fb1d2" />

Gender Analysis

Child labour differs by gender. Male children tend to have higher participation rates:

    df_long = new_df.melt(value_vars=['Sex_pct_Male', 'Sex_pct_Female'],

                       var_name='Sex', value_name='Value').dropna()
                       
    df_long['Value'] = pd.to_numeric(df_long['Value'], errors='coerce')

    sns.boxplot(df_long, x='Sex', y='Value')

    plt.show()

<img width="522" height="441" alt="image" src="https://github.com/user-attachments/assets/22c6efed-4836-40d5-b001-4f1f02d018c2" />

7. Database Integration

Creation of a PostgreSQL database called seminar, a table called child_labour, and uploading the cleaned data for scalable analysis:

    import psycopg2
    from sqlalchemy import create_engine

8.## Connect to PostgreSQL using psycopg2
### This connects to the server (not a specific database yet)
    connection = psycopg2.connect(
        host='localhost',
        user='postgres',
        password='password'
    )
    

9. ## Enable autocommit so database creation works
        connection.set_isolation_level(psycopg2.extensions.ISOLATION_LEVEL_AUTOCOMMIT)

10. ## Create a cursor to execute SQL commands
        cur = connection.cursor()

11. ## Create a new database called 'seminar'
        cur.execute('CREATE DATABASE seminar')

12. ## Close the cursor and connection
        cur.close()
        connection.close()

13. ## Create a SQLAlchemy engine connected to the 'seminar' database
## This engine is used for higher-level database operations
    engine = create_engine(
        "postgresql+psycopg2://postgres:xxx@xxx:xxx/seminar"
    )

14. ## Upload the DataFrame to PostgreSQL as a table called 'child_labour'
## If the table already exists, it will be replaced
    new_df.to_sql(
        name='child_labour',
        con=engine,
        if_exists='replace',
        index=False
    )


This setup ensures that the dataset is centralised, clean, and ready for further queries or dashboards.

8. Key Findings

Child labour rates vary significantly by region. African countries generally have the highest rates, while Central Europe has the lowest.

In 2008, South Asia and South America saw a spike due to Bolivia and Nepal.

Gender differences exist, with boys more likely to be engaged in child labour than girls.

9. Conclusion

This project demonstrates:

Data Cleaning & Standardisation: Handling messy Excel files, renaming headers, and formatting dates.

Geographic & Temporal Analysis: Mapping countries to regions and analysing trends over time.

Database Integration: Using PostgreSQL for secure storage and scalable analysis.

Insight Generation: Highlighting regional, gender, and temporal patterns in child labour worldwide.
