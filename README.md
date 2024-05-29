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

