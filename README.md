# Association-between-NO2-and-Human-Mobility
## Author: Sheida Habibi
## Title
Investigating the association between the different mobility metrics and NO2 in theair.
## Content table
|  Number  |    Content  |
|-----|-----|
|1|  [ Description ](#desc)   |
|2|   [ Summary](#meth)   |

<a name="desc"></a>
# 1. Description
This repository focuses on data preprocessing, visualization, and analysis of the relationship between Google Mobility data and NO2 levels. Specifically, it aims to explore how well Google Mobility data, along with the day of the week, can explain the variability in NO2 levels in counties with more than 500K population.

<a name="meth"></a>
# 2. Summary
The work involves obtaining data on NO2, Google mobility data, and population data, and merging them on their primary keys, which consist of the date, county name, and state name. The collected data is then preprocessed. Data cleaning, outlier handling, and missing values are addressed. Then, counties with populations greater than 500,000 are filtered for further visualization. Based on the visualization, data from mid-March to mid-April is observed to experience the same decline in NO2 levels as was seen in the same period in 2020. Therefore, a linear model is built to compare the data for the same period in 2020, 2021, and 2022. A random forest model is also used on the data to ensure that our results from the linear model are reliable. Additionally, k-fold cross-validation is employed to ensure that the linear model is not overfitting.

<a name="dg"></a>
# 3 Data Gathering and Prepration
Data Sources: Google Mobility data from google website, NO2 emission from EPA website along with the population information
Independent Variables: Google Mobility data metrics and the day of the week.
Response Variable: NO2 levels.

<a name="ld"></a>
# 3.1.1 Importing Datasets

**Libraries:**
In This project different libraries are being used. Some packages that are use in the preprocessing step is imported here:

```python
import pandas as pd
import seaborn as sns
import matplotlib.dates as mdates

```


**All data sets are read and converted to a data frame format from CSV files using **pandas**:**

**NO2 data for the years 2020, 2021, and 2022 is read using pandas :**
```python
data1 = pd.read_csv('daily_42602_2020.csv')
data2 = pd.read_csv('daily_42602_2021.csv')
data3 = pd.read_csv('daily_42602_2022.csv')
```

**No2 data of those 3 years are concatinated and added to each other:**
```python 
x = pd.concat([data1, data2], axis=0)

d_p= pd.concat([x, data3], axis=0)
```
**The required columns are selected for furthur analysis:**
```python 
d_p=d_p[['Date Local','State Name','County Name','Arithmetic Mean']]

# Convert 'Date Local' to datetime format
d_p['Date Local'] = pd.to_datetime(d_p['Date Local'])

#Taking average of NO2 for each day and each county:
d_p = d_p.groupby(['Date Local', 'State Name', 'County Name'])['Arithmetic Mean'].mean().reset_index()

# Rename the aggregated column for clarity:
d_p.rename(columns={'Arithmetic Mean': 'NO2'}, inplace=True)
d_p
```
<img width="322" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/721086a0-95e2-48f9-a527-80aff8adb63f">

**Mobility data for the years 2020, 2021, and 2022 is read, concatinated using pandas, and then needed columns are selected for furthur analysis :**
```python 
d1 = pd.read_csv('2020_US_Region_Mobility_Report.csv')
d2 = pd.read_csv('2021_US_Region_Mobility_Report.csv')
d3 = pd.read_csv('2022_US_Region_Mobility_Report.csv')

d4= pd.concat([d1, d2], axis=0)

d5= pd.concat([d4, d3], axis=0)

d_m=d5[['sub_region_1','sub_region_2','date', 'retail_and_recreation_percent_change_from_baseline', 'grocery_and_pharmacy_percent_change_from_baseline',	'parks_percent_change_from_baseline',	'transit_stations_percent_change_from_baseline' ,	'workplaces_percent_change_from_baseline',	'residential_percent_change_from_baseline' ]]
```

**Imputing the missing values by explaining them with the mean of that column:**
```python
d_m['retail_and_recreation_percent_change_from_baseline'] = d_m['retail_and_recreation_percent_change_from_baseline'].fillna(d_m['retail_and_recreation_percent_change_from_baseline'].mean())
d_m['grocery_and_pharmacy_percent_change_from_baseline'] = d_m['grocery_and_pharmacy_percent_change_from_baseline'].fillna(d_m['grocery_and_pharmacy_percent_change_from_baseline'].mean())
d_m['parks_percent_change_from_baseline'] = d_m['parks_percent_change_from_baseline'].fillna(d_m['parks_percent_change_from_baseline'].mean())
d_m['transit_stations_percent_change_from_baseline'] = d_m['transit_stations_percent_change_from_baseline'].fillna(d_m['transit_stations_percent_change_from_baseline'].mean())
d_m['workplaces_percent_change_from_baseline'] = d_m['workplaces_percent_change_from_baseline'].fillna(d_m['workplaces_percent_change_from_baseline'].mean())
d_m['residential_percent_change_from_baseline'] = d_m['residential_percent_change_from_baseline'].fillna(d_m['residential_percent_change_from_baseline'].mean())
d_m

```


**Combining NO2 and Mobility Data:(Merging on the primary key which is the vombinamtion of Date, County and State Name)**
```python
# Changing the name of dataframes for easier use
no2_df=d_p
mobility_df=d_m

# Adjust the date columns in both dataframes to ensure they are in datetime format for accurate merging
no2_df['Date Local'] = pd.to_datetime(no2_df['Date Local'])
mobility_df['date'] = pd.to_datetime(mobility_df['date'])

# Rename the columns in the mobility dataset to match those in the pollution dataset for a consistent merge:
mobility_df.rename(columns={'date': 'Date Local', 'sub_region_1': 'State Name', 'sub_region_2': 'County Name'}, inplace=True)
mobility_df['County Name'] = mobility_df['County Name'].str.replace(' County', '', regex=False)
mobility_df['County Name'] = mobility_df['County Name'].str.replace(' County', '', regex=False)


# Perform the merge (inner join) on the corrected date, county, and state columns
corrected_merged_df = pd.merge(no2_df, mobility_df, how='inner', on=['Date Local', 'State Name', 'County Name'])

# Display the first few rows f merged data:
corrected_merged_df.head()
```

**Adding Population data to our merged data:**
```python
df = pd.read_csv('cc-est2022-agesex-all.csv')
# Remove the word "County" from the 'CTYNAME' column
df['CTYNAME'] = df['CTYNAME'].str.replace(' County', '', regex=False)


# Group by county and state and calculate the mean of POPESTIMATE(because the population in a couple of consecutive years is measured and we take average as population:
grouped_mean = df.groupby(['CTYNAME', 'STNAME'])['POPESTIMATE'].mean().reset_index()
grouped_mean.rename(columns={'CTYNAME': 'County Name', 'STNAME': 'State Name'}, inplace=True)
grouped_mean
```
**Merging population data with our previous data by doing left join on the combination of county and state:**
```python
data = pd.merge(corrected_merged_df, grouped_mean, how='left', on=['State Name', 'County Name'])
```
**Categorizing the data into three categories of the population that they are located in:** 
```python
data['Date Local'] = pd.to_datetime(data['Date Local'])
data.set_index('Date Local', inplace=True)

Counties_less_100000=data[data['POPESTIMATE']<=100000]
Counties_between_100000_500000=data[(data['POPESTIMATE']>100000) & (data['POPESTIMATE']<500000)]
Counties_over_500000=data[data['POPESTIMATE']>=500000]
# Resetting the index:
Counties_over_500000.reset_index(inplace=True)
```
**Aggregate data weekly toreduce the noise and to be able to gain some information basd on the visualization:**
```python
# Calculate weekly average for specified columns
Counties_over_500000['Date Local'] = pd.to_datetime(Counties_over_500000['Date Local'])
Counties_over_500000.set_index('Date Local', inplace=True)
weekly_means = Counties_over_500000[['NO2','retail_and_recreation_percent_change_from_baseline', 'grocery_and_pharmacy_percent_change_from_baseline',	'parks_percent_change_from_baseline',	'transit_stations_percent_change_from_baseline' ,	'workplaces_percent_change_from_baseline',	'residential_percent_change_from_baseline' ]].resample('W').mean()

weekly_means.index = pd.to_datetime(weekly_means.index)
```


### Visualizing weekly average of NO2 for counties with more than 500K population:(It is worth mentioning that the visualization is for the period in which we had the information for mobility as well since we did the inner join.)
```python
# Set the aesthetic style of the plots
sns.set_style("whitegrid")

# Create a figure and set its size
plt.figure(figsize=(14, 6))

# Loop through each year and plot it
for year in [2020, 2021, 2022]:
    # Select data for the year
    yearly_data = weekly_means[weekly_means.index.year == year]
    
    # Normalize dates to a common year (e.g., 2020)
    normalized_dates = pd.to_datetime(yearly_data.index.strftime('2020-%m-%d'))
    
    # Plotting
    plt.plot(normalized_dates, yearly_data['NO2'], label=f'NO2 {year}')

# Adding title and labels
plt.title('Weekly Average NO2 Trends by Year-Counties with more than 500K Population')
plt.xlabel('Date')
plt.ylabel('NO2')

# Format the x-axis to show month names
plt.gca().xaxis.set_major_locator(mdates.MonthLocator())  # Locate months
plt.gca().xaxis.set_major_formatter(mdates.DateFormatter('%B'))  # Format as month names
# Adding a legend to distinguish between different years
plt.legend()

# Rotating the x-axis labels for better readability
plt.xticks(rotation=45)
```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/901d8c3c-2f52-411b-bfb4-1725b2129e81)



**Visualizing all the mobility variables over time in the same graph:**
```python
plt.figure(figsize=(14, 6))

# Plotting each mobility category with a unique label and line style
plt.plot(weekly_means.index, weekly_means['retail_and_recreation_percent_change_from_baseline'], label='Retail & Recreation')
plt.plot(weekly_means.index, weekly_means['grocery_and_pharmacy_percent_change_from_baseline'], label='Grocery & Pharmacy')
plt.plot(weekly_means.index, weekly_means['parks_percent_change_from_baseline'], label='Parks')
plt.plot(weekly_means.index, weekly_means['transit_stations_percent_change_from_baseline'], label='Transit Stations')
plt.plot(weekly_means.index, weekly_means['workplaces_percent_change_from_baseline'], label='Workplaces')
plt.plot(weekly_means.index, weekly_means['residential_percent_change_from_baseline'], label='Residential', linestyle='--')

# Adding title and labels
plt.title('Weekly Average Percent Change from Baseline in Mobility Trends (2020-2022)')
plt.xlabel('Date')
plt.ylabel('Percent Change from Baseline')

# Adding a legend to distinguish between different lines
plt.legend()

# Setting the locator and formatter for the x-axis
locator = mdates.MonthLocator()
formatter = mdates.DateFormatter('%b %Y')

# Apply the locator and formatter to the x-axis
ax = plt.gca()
ax.xaxis.set_major_locator(locator)
ax.xaxis.set_major_formatter(formatter)

# Rotating the x-axis labels
plt.xticks(rotation=45)

# Adjust layout for better fit and display the plot
plt.tight_layout()
plt.show()
```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/2089b67d-419c-4677-8b70-15f75f559ec5)

**Comparing parks percentage change from the baseline for different years:**
```python
weekly_means.index = pd.to_datetime(weekly_means.index)

# Set the aesthetic style of the plots
sns.set_style("whitegrid")

# Create a figure and set its size
plt.figure(figsize=(14, 6))

# Loop through each year and plot it
for year in [2020, 2021, 2022]:
    # Select data for the year
    yearly_data = weekly_means[weekly_means.index.year == year]
    
    # Normalize dates to a common year (e.g., 2020)
    normalized_dates = pd.to_datetime(yearly_data.index.strftime('2020-%m-%d'))
    
    # Plotting
    plt.plot(normalized_dates, yearly_data['parks_percent_change_from_baseline'], label=f'parks_percent_change_from_baseline {year}')

# Adding title and labels
plt.title('Weekly Average Percent Change from Baseline in Mobility Trends by Year')
plt.xlabel('Date')
plt.ylabel('Percent Change from Baseline')

# Format the x-axis to show month names
plt.gca().xaxis.set_major_locator(mdates.MonthLocator())  # Locate months
plt.gca().xaxis.set_major_formatter(mdates.DateFormatter('%B'))  # Format as month names
# Adding a legend to distinguish between different years
plt.legend()

# Rotating the x-axis labels
plt.xticks(rotation=45)
# Adjust layout for better fit and display the plot
plt.tight_layout()
plt.show()
```

![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/886d6a44-8cfe-4e46-8ecc-0e7cd4cc6d6a)

**Comparing gerocery andpharmecy percentage change from the baseline for different years:**
```python
weekly_means.index = pd.to_datetime(weekly_means.index)

# Set the aesthetic style of the plots
sns.set_style("whitegrid")

# Create a figure and set its size
plt.figure(figsize=(14, 6))

# Loop through each year and plot it
for year in [2020, 2021, 2022]:
    # Select data for the year
    yearly_data = weekly_means[weekly_means.index.year == year]
    
    # Normalize dates to a common year (e.g., 2020)
    normalized_dates = pd.to_datetime(yearly_data.index.strftime('2020-%m-%d'))
    
    # Plotting
    plt.plot(normalized_dates, yearly_data['grocery_and_pharmacy_percent_change_from_baseline'], label=f'grocery_and_pharmacy_percent_change_from_baseline {year}')

# Adding title and labels
plt.title('Weekly Average Percent Change from Baseline in Mobility Trends by Year')
plt.xlabel('Date')
plt.ylabel('Percent Change from Baseline')

# Format the x-axis to show month names
plt.gca().xaxis.set_major_locator(mdates.MonthLocator())  # Locate months
plt.gca().xaxis.set_major_formatter(mdates.DateFormatter('%B'))  # Format as month names
# Adding a legend to distinguish between different years
plt.legend()

# Rotating the x-axis labels
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/5527d9a3-b101-4795-9757-0e2bf3d86ccc)

**Comparing retail and recreation percentage change from the baseline for different years:**
```python
sns.set_style("whitegrid")

# Create a figure and set its size
plt.figure(figsize=(14, 6))

# Loop through each year and plot it
for year in [2020, 2021, 2022]:
    # Select data for the year
    yearly_data = weekly_means[weekly_means.index.year == year]
    
    # Normalize dates to a common year (e.g., 2020)
    normalized_dates = pd.to_datetime(yearly_data.index.strftime('2020-%m-%d'))
    
    # Plotting
    plt.plot(normalized_dates, yearly_data['retail_and_recreation_percent_change_from_baseline'], label=f'retail_and_recreation_percent_change_from_baseline {year}')

# Adding title and labels
plt.title('Weekly Average Percent Change from Baseline in Mobility Trends by Year')
plt.xlabel('Date')
plt.ylabel('Percent Change from Baseline')

# Format the x-axis to show month names
plt.gca().xaxis.set_major_locator(mdates.MonthLocator())  # Locate months
plt.gca().xaxis.set_major_formatter(mdates.DateFormatter('%B'))  # Format as month names
# Adding a legend to distinguish between different years
plt.legend()

# Rotating the x-axis labels
plt.xticks(rotation=45)

plt.tight_layout()
plt.show()
```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/60234806-c994-4582-a48f-6e75cf5bfeba)


**Comparing transit station percentage change from the baseline for different years:**
```python
plt.figure(figsize=(14, 6))

# Loop through each year and plot it
for year in [2020, 2021, 2022]:
    # Select data for the year
    yearly_data = weekly_means[weekly_means.index.year == year]
    
    # Normalize dates to a common year (e.g., 2020)
    normalized_dates = pd.to_datetime(yearly_data.index.strftime('2020-%m-%d'))
    
    # Plotting
    plt.plot(normalized_dates, yearly_data['transit_stations_percent_change_from_baseline'], label=f'transit_stations_percent_change_from_baselin {year}')
    
# Adding title and labels
plt.title('Weekly Average Percent Change from Baseline in Mobility Trends by Year')
plt.xlabel('Month of year')
plt.ylabel('Percent Change from Baseline')

# Format the x-axis to show month names
plt.gca().xaxis.set_major_locator(mdates.MonthLocator())  # Locate months
plt.gca().xaxis.set_major_formatter(mdates.DateFormatter('%B'))  # Format as month names
# Adding a legend to distinguish between different years
plt.legend()

# Rotating the x-axis labels
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/20b64065-4ec8-4ff9-820e-b5a659523c92)

**Comparing workplace percentage change from the baseline for different years:**
```python
sns.set_style("whitegrid")

# Create a figure and set its size
plt.figure(figsize=(14, 6))

# Loop through each year and plot it
for year in [2020, 2021, 2022]:
    # Select data for the year
    yearly_data = weekly_means[weekly_means.index.year == year]

    # Normalize dates to a common year (e.g., 2020)
    normalized_dates = pd.to_datetime(yearly_data.index.strftime('2020-%m-%d'))
    
    # Plotting
    plt.plot(normalized_dates, yearly_data['workplaces_percent_change_from_baseline'], label=f'workplaces_percent_change_from_baseline  {year}')

# Adding title and labels
plt.title('Weekly Average Percent Change from Baseline in WorkPlaces Trends by Year')
plt.xlabel('Date')
plt.ylabel('Percent Change from Baseline')

# Format the x-axis to show month names
plt.gca().xaxis.set_major_locator(mdates.MonthLocator())  # Locate months
plt.gca().xaxis.set_major_formatter(mdates.DateFormatter('%B'))  # Format as month names
# Adding a legend to distinguish between different years
plt.legend()
# Rotating the x-axis labels for better readability
plt.xticks(rotation=45)
# Adjust layout for better fit and display the plot
plt.tight_layout()
plt.show()
```

![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/328a0595-508a-455c-a190-224d8efd4a9e)

**Comparing residential percentage change from the baseline for different years:**
```python
sns.set_style("whitegrid")
# Create a figure and set its size
plt.figure(figsize=(14, 6))

# Loop through each year and plot it
for year in [2020, 2021, 2022]:
    # Select data for the year
    yearly_data = weekly_means[weekly_means.index.year == year]
    
    # Normalize dates to a common year (e.g., 2020)
    normalized_dates = pd.to_datetime(yearly_data.index.strftime('2020-%m-%d'))
    
    # Plotting
    plt.plot(normalized_dates, yearly_data['residential_percent_change_from_baseline'], label=f'residential_percent_change_from_baseline {year}')

# Adding title and labels
plt.title('Weekly Average Percent Change from Baseline in Mobility Trends by Year')
plt.xlabel('Date')
plt.ylabel('Percent Change from Baseline')

# Format the x-axis to show month names
plt.gca().xaxis.set_major_locator(mdates.MonthLocator())  # Locate months
plt.gca().xaxis.set_major_formatter(mdates.DateFormatter('%B'))  # Format as month names
# Adding a legend to distinguish between different years
plt.legend()

# Rotating the x-axis labels for better readability
plt.xticks(rotation=45)

# Adjust layout for better fit and display the plot
plt.tight_layout()
plt.show()
```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/73728c96-380e-4d8b-a7a4-c5c0fb3d9746)


**Note:From the visualization, the period of mid-March to mid-April is selected for further analysis**


**Filteringmid-March to mid-April data for the year 2020, and putting it in a CSV file:**
```python
start_date = '2020-03-15'
end_date = '2020-04-15'

df_1 = df[(df['Date Local'] >=start_date ) & (df['Date Local'] <= end_date)]


df_1['Day_of_Week'] = df_1['Date Local'].dt.dayofweek
df_1.to_csv('Counties_over_500000_2020_march_april.csv')
```

**Filteringmid-March to mid-April data for the year 2021, and putting it in a CSV file:**
```python
start_date = '2021-03-15'
end_date = '2021-04-15'

df_2 = df[(df['Date Local'] >=start_date ) & (df['Date Local'] <= end_date)]
df_2.reset_index(inplace=True)

df_2['Day_of_Week'] = df_2['Date Local'].dt.dayofweek
df_2.to_csv('Counties_over_500000_2021_march_april.csv')
```

**Filteringmid-March to mid-April data for the year 2022, and putting it in a CSV file:**
```python
start_date = '2022-03-15'
end_date = '2022-04-15'

df_3 = df[(df['Date Local'] >=start_date ) & (df['Date Local'] <= end_date)]
df_3.reset_index(inplace=True)

df_3['Day_of_Week'] = df_3['Date Local'].dt.dayofweek
df_3.to_csv('Counties_over_500000_2022_march_april.csv')
```
**Read data that became ready from Python for the year 2020 for counties with more than 500,000 population from mid-March to mid-April:**

**2020**
```{r}
data_2020 <- read.csv(file = "Counties_over_500000_2020_march_april.csv")

#converting day of week to factor so that R can consider it as categorical variable:
data_2020$Day_of_Week <- factor(data_2020$Day_of_Week)
```
**Data Summary:**
```{r}
summary(data_2020)
```
<img width="575" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/4a2fedf3-bdea-43cf-9124-144c264fcba3">

### Corrolation Matrix:
```{r, fig.height=10, fig.weight=10}
data_2020 %>% ggpairs(columns = c(3:8,2)) +
theme_bw()
```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/7b9b4a6e-746c-4281-a1a7-9c3d32f1289a)



### The correlation between No2 and the independent variables can be seen in the above matrix.From the plots it can be seen that the linear relationship exist between the dependent and independent variables

**Parks and NO2: The correlation between park mobility and NO2 is weaker (0.294), suggesting less direct impact of park visitation on NO2 levels compared to other activities.**

**Transit Stations and NO2: Transit mobility has a stronger correlation with NO2 (0.483), which aligns with the notion that increased use of transit systems might lead to higher NO2 emissions.**

**Workplaces and NO2: Workplace mobility changes have a moderate positive correlation with NO2 (0.517), implying that more people at work might contribute to increased NO2 levels, possibly due to commuting.**

**Residential and NO2: There's a negative correlation (-0.734) between residential mobility and NO2, indicating that higher stay-at-home measures might reduce NO2 levels.**

**NO2 and Recreation: The correlation between NO2 levels and recreation mobility changes is 0.482. This moderate positive correlation suggests that increased recreational activities are associated with higher NO2 levels. This could be due to increased vehicle emissions when people travel to recreational locations.**

**NO2 and Pharmacy: The correlation between NO2 levels and pharmacy mobility changes is 0.483. This moderate positive correlation indicates that higher visits to pharmacies are associated with higher NO2 levels.**


**Creating the first Model for the year 2020 including all the variables:**

```{r}

model1_2020 <- lm( NO2 ~ retail_and_recreation_percent_change_from_baseline+grocery_and_pharmacy_percent_change_from_baseline+parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+workplaces_percent_change_from_baseline+residential_percent_change_from_baseline +	Day_of_Week , data = data_2020)

# Summary
summary(model1_2020)

```
<img width="574" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/5c27c5c8-5175-4b87-84a4-3395030dc93a">

### Coefficients:
**Intercept: Estimate = 11.83834, Std. Error = 2.75077, t value = 4.304, Pr(>|t|) = 0.000383, indicating the baseline NO2 level when all predictors are zero.**

**Retail and Recreation: Estimate = 0.01069, Std. Error = 0.09012, t value = 0.119, Pr(>|t|) = 0.906865, not statistically significant.**

**Grocery and Pharmacy: Estimate = -0.04130, Std. Error = 0.06432, t value = -0.642, Pr(>|t|) = 0.528445, not statistically significant.**

**Parks: Estimate = 0.06483, Std. Error = 0.01952, t value = 3.321, Pr(>|t|) = 0.003590, statistically significant with a positive effect, indicating higher park mobility is associated with increased NO2 levels.**

**Transit Stations: Estimate = -0.30876, Std. Error = 0.14284, t value = -2.161, Pr(>|t|) = 0.043635, statistically significant with a negative effect, suggesting increased transit station mobility is associated with decreased NO2 levels.**

**Workplaces: Estimate = 0.09311, Std. Error = 0.12660, t value = 0.735, Pr(>|t|) = 0.471048, not statistically significant.**

**Residential: Estimate = -0.67997, Std. Error = 0.24151, t value = -2.816, Pr(>|t|) = 0.011043, statistically significant with a negative effect, indicating increased residential mobility (more staying at home) is associated with lower NO2 levels.**

**Day_of_Week1: Estimate = 1.37066, Std. Error = 0.74935, t value = 1.829, Pr(>|t|) = 0.08312, marginally significant.
Day_of_Week2: Estimate = 1.60718, Std. Error = 0.78347, t value = 2.052, Pr(>|t|) = 0.054276, marginally significant.
Day_of_Week3: Estimate = 1.18909, Std. Error = 0.82862, t value = 1.435, Pr(>|t|) = 0.167536, not statistically significant.
Day_of_Week4: Estimate = 1.77416, Std. Error = 1.21835, t value = 1.456, Pr(>|t|) = 0.161667, not statistically significant.
Day_of_Week5: Estimate = -5.19269, Std. Error = 1.44420, t value = -3.596, Pr(>|t|) = 0.001928, statistically significant with a negative effect.
Day_of_Week6: Estimate = -8.29014, Std. Error = 1.63449, t value = -5.072, Pr(>|t|) = 6.77e-05, statistically significant with a strong negative effect.**

### Checking the linear model conditions to see if they are met using the diagnostic plot:"**
```{r}
par(mfrow = c(2,2), oma = c(0,0,2,0))
plot(model1_2020, pch = 16, sub.caption = "")
title(main="Diagnostics for model1_2020", outer=TRUE)

```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/1f954644-b099-4467-86be-da1f6b93f6ad)

**It seems the point 1 is somehow influential since it has high leverage and the cook's D ri in the border of 1**
### excluding point 1 from the data:

```{r}
data_2020 <- data_2020%>% slice(-1)
```

**Conducting the previous model after slicing point 1:**

```{r}
model1_2020 <- lm( NO2 ~ retail_and_recreation_percent_change_from_baseline+grocery_and_pharmacy_percent_change_from_baseline+parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+workplaces_percent_change_from_baseline+residential_percent_change_from_baseline +	Day_of_Week , data = data_2020)

# Summary
summary(model1_2020)

```
# Backwars elimination:
** We do feature selection using Backward elimination method which helps in identifying the most significant predictors and simplifying the model. Therefore we will end up with a model that is easier to interpret and potentially more robust.**

### Removing retail_and_recreation_percent_change_from_baseline (p = 0.906865)- That has the highest p-value:
```{r}
model2_2020 <- lm( NO2 ~ grocery_and_pharmacy_percent_change_from_baseline+parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+workplaces_percent_change_from_baseline+residential_percent_change_from_baseline +	Day_of_Week , data = data_2020)

# Summary
summary(model2_2020)

```
<img width="575" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/08d5ae8c-7011-4905-826f-8522b2c22bc7">
### Checking for the diagnostic plot:

![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/cf65448b-2bc9-489f-90de-808b9d4a8217)


### Next step in doing backward elimination: excluding 'grocery_and_pharmacy_percent_change_from_baseline' that has comparably higher p-value than the other variables:

```{r}
model3_2020 <- lm( NO2 ~ parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+workplaces_percent_change_from_baseline+residential_percent_change_from_baseline +	Day_of_Week , data = data_2020)

# Summary
summary(model3_2020)

```
<img width="537" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/4e3235cd-9205-4522-8a47-9e4ff2a598f4">
### checking for the condition:

```{r}
par(mfrow = c(2,2), oma = c(0,0,2,0))
plot(model3_2020, pch = 16, sub.caption = "")
title(main="Diagnostics for model3_2020", outer=TRUE)

```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/effda9ae-062a-49aa-ad02-cc4720bd0bb8)




### Next step in doing backward elimination: excluding 'workplaces_percent_change_from_baseline' that has a comparably higher p-value than the other variables:

```{r}
model4_2020 <- lm( NO2 ~ parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+residential_percent_change_from_baseline +	Day_of_Week , data = data_2020)

# Summary
summary(model4_2020)

```
<img width="619" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/59de84cc-5baf-4b76-b889-5610a6d1f64a">
### Some days seem to be significant, and the others seem not. for those days that are significant, the intercept would be different

### Lets check the model:
```{r}
par(mfrow = c(2,2), oma = c(0,0,2,0))
plot(model4_2020, pch = 16, sub.caption = "")
title(main="Diagnostics for model4_2020", outer=TRUE)

```

![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/08a9c823-bb30-4100-94ac-e55232b0c2d5)

# Significant Predictors:

## Parks Mobility: 
**Significant positive effect, suggesting that increases in park mobility are associated with higher NO2 levels.**
#### For a 1 percent from baseline increase in parks_percent_change_from_baseline, we estimate the mean of NO2 percentage increase by 0.06593 after controlling for transit station, residential mobilities, and days of the week.

## Transit Stations Mobility:
**Significant negative effect, indicating that increased transit station mobility is associated with lower NO2 levels.**

#### For a 1 percent increase from baseline in transit stations percent change, we estimate the mean of NO2 percentage to decrease by 0.37814, after controlling for  parks, residential mobilities, and days of the week.


## Residential Mobility:
**Significant negative effect, showing that more time spent at home is associated with lower NO2 levels.**
#### For a 1 percent increase from baseline in residential percent change, we estimate the mean of NO2 percentage to decrease by  0.95633, after controlling for   parks, transit station mobilities, and days of the week

## Day_of_Week5 and Day_of_Week6:
**Significant negative effects, indicating that NO2 levels are lower on these days compared to the reference day**





**2021**

**2022**

**2022**

