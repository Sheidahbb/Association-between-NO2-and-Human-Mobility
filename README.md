
## Author: Sheida Habibi
## Title 
Evaluating Changes in NO2 Levels in Response to Mobility Insights from Mid-March to Mid-April, for the years 2020, 2021, and 2022(During and Post-Lockdown), Across counties with more than 500K population in the United States
## Content table
|  Number  |    Content  |
|-----|-----|
|1|  [ Description ](#desc)   |
|2|   [ Summary](#meth)   |
|3|    [ Data](#data)   |
|3.1|    [ Data Gathering and Preparation ](#dg)   |
|3.1.1|    [ Reading and Preparing the NO2 Datasets ](#dp)   |
|3.1.2|    [ Reading and Preparing the Mobility Datasets ](#mo)   |
|3.1.3|    [ Merging the NO2 and Mobility Datasets ](#me1)   |
|3.14|    [ Reading and Preparing the Population Dataset ](#po)   |
|3.1.5|    [ Final Dataset ](#me2)   |
|3.2|    [ Filtering counties with more than 500K population ](#fi)   |
|4|   [ Visualizations ](#vs)    |
|4.1|   [ NO2 Visualizations ](#vs-NO2)    |
|4.2|   [ Mobility Visualizations ](#vs-Mobility)    |
|5|   [ Filtering based on intended period ](#filtering)    |
|6|   [ Method](#model)    |
|6.2|   [ 2020](#model_2020)  |
|6.2.1|   [ 2020_Correlation Metrix](#model_2020_1)    |
|6.2.1|   [ 2020_MLR including all variables](#model_2020_2)    |
|6.2.2|   [ 2020_Backward Elimination Process](#model_2020_3)    |
|6.2.3|   [2020_FinalMLR Model ](#model_2020_4)    |
|6.2.5|   [2020_K-fold cross-Validation on MLR](#model_2020_5)    |
|6.2.6|   [ 2020_Random forest](#model_2020_6)    |
|6.2|   [ 2021](#model_2021)  |
|6.2.1|   [ 2021_Correlation Metrix](#model_2021_1)    |
|6.2.1|   [ 2021_MLR including all variables](#model_2021_2)    |
|6.2.2|   [ 2021_Backward Elimination Process](#model_2021_3)    |
|6.2.3|   [2021_FinalMLR Model ](#model_2021_4)    |
|6.2.5|   [2021_K-fold cross-Validation on MLR](#model_2021_5)    |
|6.2.6|   [ 2021_Random forest](#model_2021_6)    |
|6.3|   [ 2022](#model_2022)  |
|6.3.1|   [ 2022_Correlation Metrix](#model_2022_1)    |
|6.3.1|   [ 2022_MLR including all variables](#model_2022_2)    |
|6.3.2|   [ 2022_Backward Elimination Process](#model_2022_3)    |
|6.3.3|   [2022_FinalMLR Model ](#model_2022_4)    |
|6.3.5|   [2022_K-fold cross-Validation on MLR](#model_2022_5)    |
|6.3.6|   [ 2022_Random forest](#model_2022_6)    |
|7|   [Conclusion](#conclusion)    |
|8|   [ Limitations](#limit)    |

<a name="desc"></a>
# 1. Description

This project focuses on data preprocessing, visualization, and analysis to evaluate the relationship between Google Mobility data and NO2 levels across U.S. counties with populations exceeding 500,000. Specifically, it aims to explore how well Google Mobility data, along with the day of the week, can explain the variability in NO2 levels during the period from mid-March to mid-April for the years 2020, 2021, and 2022. 

<a name="meth"></a>
# 2. Summary
The work involves obtaining data on NO2, Google mobility data, and population data, and merging them on their primary keys, which consist of the date, county name, and state name. The collected data is then preprocessed. Data cleaning, outlier handling, and missing values are addressed. Then, counties with populations greater than 500,000 are filtered for further visualization. Based on the visualization, data from mid-March to mid-April is observed to experience the same decline in NO2 levels as was seen in the same period in 2020. Therefore, a linear model is built to compare the data for the same period in 2020, 2021, and 2022. A random forest model is also used on the data to ensure that our results from the linear model are reliable. Additionally, k-fold cross-validation is employed to ensure that the linear model is not overfitting.

<a name="data"></a>
# 3. Data Gathering and Preparation
Data Sources: Google Mobility data from the Google website, NO2 emission from the EPA website along with the population information
Independent Variables: Google Mobility data metrics and the day of the week.

## More description on Google mobility variables and how they are calculated.

| Google Mobility Variable  |    Description  |
|-----|-----|
|Retail and Recreation| Changes in visits to places like restaurants, cafes, shopping centers, theme parks, museums, libraries, and movie theaters.|
|Grocery and Pharmacy| Changes in visits to grocery stores, food warehouses, farmers markets, specialty food shops, drug stores, and pharmacies.|
|Parks| Changes in visits to national parks, public beaches, marinas, dog parks, plazas, and public gardens.|
|Transit Stations| Changes in visits to public transport hubs such as subway, bus, and train stations.|
|Workplaces| Changes in the number of people visiting workplaces|
|Residential| Changes in the amount of time people spend at home.|

The Google Mobility variables are measured as percentage changes compared to a baseline value, which represents the median value for that day of the week during the 5-week period from January 3 to February 6, 2020. The data is collected from users who have enabled Location History on their Google accounts, ensuring privacy and anonymization.
Response Variable: NO2 levels.

Libraries:
In This project different libraries are being used. Some packages that are used in the preprocessing step are imported here:

```python
import pandas as pd
import seaborn as sns
import matplotlib.dates as mdates

```

**All data sets are read and converted to a data frame format from CSV files using **pandas**:**
<a name="dp"></a>
# 3.1.1 Reading and Preparing the NO2 Datasets

**NO2 data for the years 2020, 2021, and 2022 is read using pandas:**
```python
data1 = pd.read_csv('daily_42602_2020.csv')
data2 = pd.read_csv('daily_42602_2021.csv')
data3 = pd.read_csv('daily_42602_2022.csv')
```

No2 data of those 3 years are concatenated and added to each other:
```python 
x = pd.concat([data1, data2], axis=0)

d_p= pd.concat([x, data3], axis=0)
```

The required columns are selected for further analysis:*
```python 
d_p=d_p[['Date Local','State Name','County Name','Arithmetic Mean']]

# Convert 'Date Local' to datetime format
d_p['Date Local'] = pd.to_datetime(d_p['Date Local'])

#Taking the average of NO2 for each day and each county:
d_p = d_p.groupby(['Date Local', 'State Name', 'County Name'])['Arithmetic Mean'].mean().reset_index()

# Rename the aggregated column for clarity:
d_p.rename(columns={'Arithmetic Mean': 'NO2'}, inplace=True)
d_p
```
<img width="322" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/721086a0-95e2-48f9-a527-80aff8adb63f">

<a name="mo"></a>
# 3.1.2 Reading and Preparing the Mobility Datasets

Mobility data for the years 2020, 2021, and 2022 is read and concatenated using pandas, and then needed columns are selected for further analysis:
```python 
d1 = pd.read_csv('2020_US_Region_Mobility_Report.csv')
d2 = pd.read_csv('2021_US_Region_Mobility_Report.csv')
d3 = pd.read_csv('2022_US_Region_Mobility_Report.csv')

d4= pd.concat([d1, d2], axis=0)

d5= pd.concat([d4, d3], axis=0)

d_m=d5[['sub_region_1','sub_region_2','date', 'retail_and_recreation_percent_change_from_baseline', 'grocery_and_pharmacy_percent_change_from_baseline',	'parks_percent_change_from_baseline',	'transit_stations_percent_change_from_baseline' ,	'workplaces_percent_change_from_baseline',	'residential_percent_change_from_baseline' ]]
```

Imputing the missing values by explaining them with the mean of that column:
```python
d_m['retail_and_recreation_percent_change_from_baseline'] = d_m['retail_and_recreation_percent_change_from_baseline'].fillna(d_m['retail_and_recreation_percent_change_from_baseline'].mean())
d_m['grocery_and_pharmacy_percent_change_from_baseline'] = d_m['grocery_and_pharmacy_percent_change_from_baseline'].fillna(d_m['grocery_and_pharmacy_percent_change_from_baseline'].mean())
d_m['parks_percent_change_from_baseline'] = d_m['parks_percent_change_from_baseline'].fillna(d_m['parks_percent_change_from_baseline'].mean())
d_m['transit_stations_percent_change_from_baseline'] = d_m['transit_stations_percent_change_from_baseline'].fillna(d_m['transit_stations_percent_change_from_baseline'].mean())
d_m['workplaces_percent_change_from_baseline'] = d_m['workplaces_percent_change_from_baseline'].fillna(d_m['workplaces_percent_change_from_baseline'].mean())
d_m['residential_percent_change_from_baseline'] = d_m['residential_percent_change_from_baseline'].fillna(d_m['residential_percent_change_from_baseline'].mean())
d_m

```
<a name="me1"></a>
# 3.1.3 Merging the NO2 and Mobility Datasets
Combining NO2 and Mobility Data:(Merging on the primary key which is the combination of Date, County and State Name)
```python
# Changing the name of data frames for easier use
no2_df=d_p
mobility_df=d_m

# Adjust the date columns in both dataframes to ensure they are in DateTime format for accurate merging
no2_df['Date Local'] = pd.to_datetime(no2_df['Date Local'])
mobility_df['date'] = pd.to_datetime(mobility_df['date'])

# Rename the columns in the mobility dataset to match those in the pollution dataset for a consistent merge:
mobility_df.rename(columns={'date': 'Date Local', 'sub_region_1': 'State Name', 'sub_region_2': 'County Name'}, inplace=True)
mobility_df['County Name'] = mobility_df['County Name'].str.replace(' County', '', regex=False)
mobility_df['County Name'] = mobility_df['County Name'].str.replace(' County', '', regex=False)


# Perform the merge (inner join) on the corrected date, county, and state columns
corrected_merged_df = pd.merge(no2_df, mobility_df, how='inner', on=['Date Local', 'State Name', 'County Name'])

# Display the first few rows of merged data:
corrected_merged_df.head()
```
<a name="po"></a>
# 3.1.4 Reading and Preparing the Population Dataset
Adding Population data to our merged data:
```python
df = pd.read_csv('cc-est2022-agesex-all.csv')
# Remove the word "County" from the 'CTYNAME' column
df['CTYNAME'] = df['CTYNAME'].str.replace(' County', '', regex=False)


# Group by county and state and calculate the mean of POPESTIMATE(because the population in a couple of consecutive years is measured and we take average as population:
grouped_mean = df.groupby(['CTYNAME', 'STNAME'])['POPESTIMATE'].mean().reset_index()
grouped_mean.rename(columns={'CTYNAME': 'County Name', 'STNAME': 'State Name'}, inplace=True)
grouped_mean
```
<a name="me2"></a>
# 3.1.6 Final Dataset

Merging population data with our previous data by doing left join on the combination of county and state:
```python
data = pd.merge(corrected_merged_df, grouped_mean, how='left', on=['State Name', 'County Name'])
```
<a name="fi"></a>
# 3.2  Filtering counties with more than 500K population
Categorizing the data into three categories of the population that they are located in: 
```python
data['Date Local'] = pd.to_datetime(data['Date Local'])
data.set_index('Date Local', inplace=True)

Counties_less_100000=data[data['POPESTIMATE']<=100000]
Counties_between_100000_500000=data[(data['POPESTIMATE']>100000) & (data['POPESTIMATE']<500000)]
Counties_over_500000=data[data['POPESTIMATE']>=500000]
# Resetting the index:
Counties_over_500000.reset_index(inplace=True)
```

<a name="vs1"></a>
# 4 Visualizations

Aggregate data weekly to reduce the noise and to be able to gain some information based on the visualization:
```python
# Calculate the weekly average for specified columns
Counties_over_500000['Date Local'] = pd.to_datetime(Counties_over_500000['Date Local'])
Counties_over_500000.set_index('Date Local', inplace=True)
weekly_means = Counties_over_500000[['NO2','retail_and_recreation_percent_change_from_baseline', 'grocery_and_pharmacy_percent_change_from_baseline',	'parks_percent_change_from_baseline',	'transit_stations_percent_change_from_baseline' ,	'workplaces_percent_change_from_baseline',	'residential_percent_change_from_baseline' ]].resample('W').mean()

weekly_means.index = pd.to_datetime(weekly_means.index)
```

<a name="vs-NO2"></a>
# 4.1 NO2 Visualizations vs-NO2
Visualizing weekly average of NO2 for counties with more than 500K population:(It is worth mentioning that the visualization is for the period in which we had the information for mobility as well since we did the inner join.)
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

**I did the following visualizations using Tableau which is a tool for data visualization**

Visualizing the NO2 Data for the period of mid-March to mid-April 2020:
<img width="700" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/e1ca79c0-e075-41f8-9b7f-789a658b2010">


Visualizing the NO2 Data for the period of mid-March to mid-April 2021:

<img width="700" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/6ee8d5d3-ff74-43d3-8f69-a65660788567">

Visualizing the NO2 Data for the period of mid march to mid-April 2022:

<img width="700" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/8c0c0a87-27ec-4c15-968e-7bdbe55b980e">

<a name="vs-Mobility"></a>
# 4.2. Mobility Visualizations 

Visualizing all the mobility variables over time in the same graph:

```python
plt.figure(figsize=(14, 6))
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
plt.tight_layout()
plt.show()
```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/2089b67d-419c-4677-8b70-15f75f559ec5)

Comparing parks percentage change from the baseline for different years:
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

Comparing grocery and pharmacy percentage change from the baseline for different years:
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

Comparing retail and recreation percentage change from the baseline for different years:
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


Comparing transit station percentage change from the baseline for different years:
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

Comparing workplace percentage change from the baseline for different years:
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

Comparing residential percentage change from the baseline for different years:
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

As an instance, I brought Mobility data on the map and the distribution of that for different states located in different counties:
**I did this visualization by Tableau which is a tool for data visualization.**
<img width="800" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/66c2d876-bbe2-46bf-b29c-855fbe8d0d99">

Note: From the visualization, the period of mid-March to mid-April is selected for further analysis
<a name="filtering"></a>
# 5 Filtering based on the intended period 
Filteringmid-March to mid-April data for the year 2020, and putting it in a CSV file:
```python
start_date = '2020-03-15'
end_date = '2020-04-15'

df_1 = df[(df['Date Local'] >=start_date ) & (df['Date Local'] <= end_date)]


df_1['Day_of_Week'] = df_1['Date Local'].dt.dayofweek
df_1.to_csv('Counties_over_500000_2020_march_april.csv')
```

Filteringmid-March to mid-April data for the year 2021, and putting it in a CSV file:
```python
start_date = '2021-03-15'
end_date = '2021-04-15'

df_2 = df[(df['Date Local'] >=start_date ) & (df['Date Local'] <= end_date)]
df_2.reset_index(inplace=True)

df_2['Day_of_Week'] = df_2['Date Local'].dt.dayofweek
df_2.to_csv('Counties_over_500000_2021_march_april.csv')
```

Filteringmid-March to mid-April data for the year 2022, and putting it in a CSV file:
```python
start_date = '2022-03-15'
end_date = '2022-04-15'

df_3 = df[(df['Date Local'] >=start_date ) & (df['Date Local'] <= end_date)]
df_3.reset_index(inplace=True)

df_3['Day_of_Week'] = df_3['Date Local'].dt.dayofweek
df_3.to_csv('Counties_over_500000_2022_march_april.csv')
```

<a name="model_2020"></a>
# 6.1. 2020

Reading data that became ready from Python for the year 2020 for counties with more than 500,000 population from mid-March to mid-April:*

```{r}
data_2020 <- read.csv(file = "Counties_over_500000_2020_march_april.csv")

#converting the day of week to factor so that R can consider it as a categorical variable:
data_2020$Day_of_Week <- factor(data_2020$Day_of_Week)
```
**Data Summary:**
```{r}
summary(data_2020)
```
<img width="575" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/4a2fedf3-bdea-43cf-9124-144c264fcba3">

<a name="model_2020_1"></a>
# 6.1.1 Correlation Matrix:
```{r, fig.height=10, fig.weight=10}
data_2020 %>% ggpairs(columns = c(3:8,2)) +
theme_bw()
```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/7b9b4a6e-746c-4281-a1a7-9c3d32f1289a)



### The correlation between No2 and the independent variables can be seen in the above matrix.From the plots it can be seen that the linear relationship exist between the dependent and independent variables

**Parks and NO2:** The correlation between park mobility and NO2 is weaker (0.294), suggesting less direct impact of park visitation on NO2 levels compared to other activities.**

**Transit Stations and NO2:** Transit mobility has a stronger correlation with NO2 (0.483), which aligns with the notion that increased use of transit systems might lead to higher NO2 emissions.

**Workplaces and NO2:** Workplace mobility changes have a moderate positive correlation with NO2 (0.517), implying that more people at work might contribute to increased NO2 levels, possibly due to commuting.

**Residential and NO2:** There's a negative correlation (-0.734) between residential mobility and NO2, indicating that higher stay-at-home measures might reduce NO2 levels.

** NO2 and Recreation: ** The correlation between NO2 levels and recreation mobility changes is 0.482. This moderate positive correlation suggests that increased recreational activities are associated with higher NO2 levels. This could be due to increased vehicle emissions when people travel to recreational locations.

**NO2 and Pharmacy:** The correlation between NO2 levels and pharmacy mobility changes is 0.483. This moderate positive correlation indicates that higher visits to pharmacies are associated with higher NO2 levels.

<a name="model_2020_2"></a>
# 6.3.2 Complete MLR:
Here is the Multiple Linear Regression Model for 2020 data from mid-March to mid-April for counties with more than 500K population(Considering all the mobility variables as well as the day of the week as the independent variable).

```{r}
model1_2020 <- lm( NO2 ~ retail_and_recreation_percent_change_from_baseline+grocery_and_pharmacy_percent_change_from_baseline+parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+workplaces_percent_change_from_baseline+residential_percent_change_from_baseline +	Day_of_Week , data = data_2020)

# Summary
summary(model1_2020)

```
<img width="574" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/5c27c5c8-5175-4b87-84a4-3395030dc93a">

### Coefficients:
**Intercept:** Estimate = 11.83834, Std. Error = 2.75077, t value = 4.304, Pr(>|t|) = 0.000383, indicating the baseline NO2 level when all predictors are zero.

**Retail and Recreation:** Estimate = 0.01069, Std. Error = 0.09012, t value = 0.119, Pr(>|t|) = 0.906865, not statistically significant.**

**Grocery and Pharmacy:** Estimate = -0.04130, Std. Error = 0.06432, t value = -0.642, Pr(>|t|) = 0.528445, not statistically significant.

**Parks:** Estimate = 0.06483, Std. Error = 0.01952, t value = 3.321, Pr(>|t|) = 0.003590, statistically significant with a positive effect, indicating higher park mobility is associated with increased NO2 levels.**

**Transit Stations:** Estimate = -0.30876, Std. Error = 0.14284, t value = -2.161, Pr(>|t|) = 0.043635, statistically significant with a negative effect, suggesting increased transit station mobility is associated with decreased NO2 levels.**

**Workplaces:** Estimate = 0.09311, Std. Error = 0.12660, t value = 0.735, Pr(>|t|) = 0.471048, not statistically significant.

**Residential:** Estimate = -0.67997, Std. Error = 0.24151, t value = -2.816, Pr(>|t|) = 0.011043, statistically significant with a negative effect, indicating increased residential mobility (more staying at home) is associated with lower NO2 levels.

**Day_of_Week1: Estimate = 1.37066, Std. Error = 0.74935, t value = 1.829, Pr(>|t|) = 0.08312, marginally significant.
Day_of_Week2: Estimate = 1.60718, Std. Error = 0.78347, t value = 2.052, Pr(>|t|) = 0.054276, marginally significant.
Day_of_Week3: Estimate = 1.18909, Std. Error = 0.82862, t value = 1.435, Pr(>|t|) = 0.167536, not statistically significant.
Day_of_Week4: Estimate = 1.77416, Std. Error = 1.21835, t value = 1.456, Pr(>|t|) = 0.161667, not statistically significant.
Day_of_Week5: Estimate = -5.19269, Std. Error = 1.44420, t value = -3.596, Pr(>|t|) = 0.001928, statistically significant with a negative effect.
Day_of_Week6: Estimate = -8.29014, Std. Error = 1.63449, t value = -5.072, Pr(>|t|) = 6.77e-05, statistically significant with a strong negative effect.**

Checking the linear model conditions to see if they are met using the diagnostic plot:
```{r}
par(mfrow = c(2,2), oma = c(0,0,2,0))
plot(model1_2020, pch = 16, sub.caption = "")
title(main="Diagnostics for model1_2020", outer=TRUE)

```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/1f954644-b099-4467-86be-da1f6b93f6ad)

**It seems that point 1 is somehow influential since it has high leverage and the cook's D ri in the border of 1**
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

## Kfold cross-validation 
** This step is implemented to evaluate the performance of a machine learning model and ensure that it is not overfitting.**
```{r}
set.seed(123)
library(caret)
data_2020 = data_2020

# Define control method for 4-fold CV
control <- trainControl(method = "cv", number = 4)

# Train the model with linear regression
model_kfold_2020 <- train(NO2 ~ parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+residential_percent_change_from_baseline +	Day_of_Week  , data = data_2020, method = "lm", trControl = control)

# Print the results, including RMSE
print(model_kfold_2020)
```

<img width="314" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/2c85b4d2-24f9-49d9-987b-72bb85b9a67c">

## Random Foretst:
```{r}

# Creating indices for the train set
trainIndex <- createDataPartition(data_2020$NO2, p = 0.75, list = FALSE, times = 1)

# Create the training data and testing data
trainData <- data_2020[trainIndex, ]
testData <- data_2020[-trainIndex, ]

library(randomForest)
# Create the random forest model
rf_model_2020 <- randomForest(NO2 ~  parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+residential_percent_change_from_baseline +	Day_of_Week , data = trainData, ntree = 100, mtry = 3, importance = TRUE)


print(rf_model_2020)

```

<img width="579" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/2cc4e2b0-4f60-40aa-85fa-caa467481b24">

```{r}
# Predict using the random forest model
predictions <- predict(rf_model_2020, testData)



mse <- mean((predictions - testData$NO2)^2)
rmse <- sqrt(mse)
mae <- mean(abs(testData$NO2 - predictions))
print(paste("Mean Absolute Error (MAE):", mae))
print(paste("MSE:", mse))
print(paste("RMSE:", rmse))

```

<img width="478" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/4167a176-07b8-45af-8649-3adf22099006">


<a name="model_2021"></a>
# 6.2. 2021

### Reading Data
```{r}
data_2021 <- read.csv(file = "Counties_over_500000_2021_march_april.csv")

data_2021$Day_of_Week <- factor(data_2021$Day_of_Week)

```


```{r}
data_2021<- select(data_2021,c( -X,-index))
```
<a name="model_2022_1"></a>
### Correlation metric:
```{r, fig.height=10, fig.weight=10}
data_2021 %>% ggpairs(columns = c(3:8,2)) +
theme_bw()

```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/643c33dd-b821-4ac0-99a2-a5c023d5e086)

<a name="model_2022_2"></a>
# 6.3.2 Complete MLR:

### Here is the Multiple Linear Regression Model, including all the variables for 2021 data from mid-March to mid-April for counties with more than 500K population(Considering all the mobility variables as well as the day of the week as the independent variable).

```{r}

model1_2021 <- lm( NO2 ~ retail_and_recreation_percent_change_from_baseline+grocery_and_pharmacy_percent_change_from_baseline+parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+workplaces_percent_change_from_baseline+residential_percent_change_from_baseline +	Day_of_Week , data = data_2021)

# Summary
summary(model1_2021)

```
<img width="594" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/7eb268a3-1b45-4cbe-99e7-a9c8fbe58833">

### Lets's check the model first:
```{r}
par(mfrow = c(2,2), oma = c(0,0,2,0))
plot(model1_2021, pch = 16, sub.caption = "")
title(main="Diagnostics for model1_2021", outer=TRUE)

```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/31ae67ea-0d1e-4ff3-b01c-9dda0e0fac70)

### As can be seen, point 21 is influential and violates the LR condition; therefore, we need to exclude it.

## Excluding point 21 which is an influential point:
```{r}
data_2021 = data_2021%>% slice(-21)
```

<a name="model_2022_3"></a>
# 6.3.1 Backward elimination:

**We do feature selection using the Backward elimination method, which helps in identifying the most significant predictors and simplifying the model. Therefore we will end up with a model that is easier to interpret and potentially more robust.**

### Let's exclude residential_percent_change_from_baseline from the model since it is the least significant.

```{r}

model2_2021 <- lm( NO2 ~ retail_and_recreation_percent_change_from_baseline+grocery_and_pharmacy_percent_change_from_baseline+parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+workplaces_percent_change_from_baseline +	Day_of_Week , data = data_2021)

# Summary
summary(model2_2021)

```
<img width="605" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/2627a583-c8e3-4798-845b-47cc09dea67a">

### Let's exlude the day of week since it is the least significant:
```{r}

model3_2021 <- lm( NO2 ~ retail_and_recreation_percent_change_from_baseline+grocery_and_pharmacy_percent_change_from_baseline+parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+workplaces_percent_change_from_baseline  , data = data_2021)

# Summary
summary(model3_2021)

```
<img width="614" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/880534c6-2a7f-46ff-87a1-5cb3d169fa2c">


### Let's exclude grocery_and_pharmacy_percent_change_from_baseline since it is the least significant:

```{r}

model4_2021 <- lm( NO2 ~ retail_and_recreation_percent_change_from_baseline+parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+workplaces_percent_change_from_baseline  , data = data_2021)

# Summary
summary(model4_2021)

```
<img width="566" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/8054d147-6008-42f3-9a14-ad17b4e5be39">

```{r}
par(mfrow = c(2,2), oma = c(0,0,2,0))
plot(model3_2021, pch = 16, sub.caption = "")
title(main="Diagnostics for m4", outer=TRUE)

```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/7e12f994-e440-4da4-8646-8f31a202968c)

### The model seems good since all the variables seems significant.

### Seems gtreat! based on the model that we have now, we can say that the following variables play a significant role in explaining the NO2 for the period of mid marrch to mid april for the year 2021.

### 1.parks_percent_change_from_baseline
### 2.transit_stations_percent_change_from_baseline
### 3.residential_percent_change_from_baseline 
### 4.workplaces_percent_change_from_baseline



## Final Model 2021:

#### Now, we can talk about the result of model 4 as our final model for year 2021. We have a good evidence that all the remaining variables have impact on the model as the p values are small. Also, we met all the linear model condition along the way.

#### NO2 = 0.44173 + 0.29639 × (retail_and_recreation_percent_change_from_baseline) + 0.04553 × (parks_percent_change_from_baseline) − 0.53085 × (transit_stations_percent_change_from_baseline) + 0.09487 × (workplaces_percent_change_from_baseline)

## Interpratation of MLR Model4:


#### Retail and Recreation: Each 1% increase from the baseline is associated with a 0.29639 increase in NO2 levels.

#### Parks: Each 1% increase from the baseline is associated with a 0.04553 increase in NO2 levels.

#### Transit Stations: Each 1% increase from the baseline is associated with a 0.53085 decrease in NO2 levels.

#### Workplaces: Each 1% increase from the baseline is associated with a 0.09487 increase in NO2 levels.


### K fold cross validation:
```{r}

# Define control method for 4-fold CV
control <- trainControl(method = "cv", number = 4)

# Train the model with linear regression
# Replace y with your target variable, and . indicates all other variables as predictors
model_kfold_2021 <- train(NO2 ~ retail_and_recreation_percent_change_from_baseline+parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+workplaces_percent_change_from_baseline, data = data_2021, method = "lm", trControl = control)

# Print the results, including RMSE
print(model_kfold_2021)

```
<img width="438" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/b09793f7-b300-4c23-b498-10992bc931c4">

# Let's performe random forest to compare with our model to make sure that the random forest is performing okay

```{r}

# Set the seed for reproducibility
set.seed(123)


# Create indices for the train set
trainIndex <- createDataPartition(data_2021$NO2, p = 0.75, list = FALSE, times = 1)

# Create the training data and testing data
trainData <- data_2021[trainIndex, ]
testData <- data_2021[-trainIndex, ]


library(randomForest)
# Create the random forest model
# y is the numeric response variable and . represents all other variables in the data as predictors
rf_model_2021 <- randomForest(NO2 ~ retail_and_recreation_percent_change_from_baseline+parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+workplaces_percent_change_from_baseline, data = trainData, ntree = 100, mtry = 3, importance = TRUE)

# Print the model summary
print(rf_model_2021)

# Plot error as trees are added
plot(rf_model_2021)

# View variable importance
importance(rf_model_2021)
varImpPlot(rf_model_2021)

```
<img width="526" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/1d3ff442-5e09-4ca4-b240-5f4231f00c67">

```{r}
# Predict using the random forest model
predictions <- predict(rf_model_2021, testData)

mse <- mean((predictions - testData$NO2)^2)
rmse <- sqrt(mse)
mae <- mean(abs(testData$NO2 - predictions))
print(paste("Mean Absolute Error (MAE):", mae))
print(paste("MSE:", mse))
print(paste("RMSE:", rmse))
```
<img width="397" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/2e9b4342-3f66-4a49-930e-9a0458146bca">

**Comparing the results, we can say that linear regression is a good model for explaining NO2.***

<a name="model_2022"></a>
# 2022

### Reading Data:
```{r}
data_2022 <- read.csv(file = "Counties_over_500000_2022_march_april.csv")
data_2022$Day_of_Week <- factor(data_2022$Day_of_Week)

```


```{r}
data_2022<- select(data_2022, c(-X,-index))
```
<a name="model_2022_1"></a>
# 6.3.1 Correlation Matrix:

```{r, fig.height=10, fig.weight=10}
data_2022 %>% ggpairs(columns = c(3:8,2)) +
theme_bw()

```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/6433e0b2-4a3e-4478-b91a-7f4dd698aa68)

<a name="model_2022_2"></a>
# 6.3.2 Complete MLR:

### Here is the Multiple Linear Regression Model, including all the variables, for 2022 data from mid-March to mid-April for counties with more than 500K population(Considering all the mobility variables as well as the day of the week as the independent variable).
```{r}
model1_2022 <- lm( NO2 ~ retail_and_recreation_percent_change_from_baseline+grocery_and_pharmacy_percent_change_from_baseline+parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+workplaces_percent_change_from_baseline+residential_percent_change_from_baseline +	Day_of_Week , data = data_2022)

# Summary
summary(model1_2022)

```
<img width="593" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/d3b82cb5-6a85-4cf6-8c16-3b129cc89a6a">

Checking for the conditions using the diagnostic plot:
```{r}
par(mfrow = c(2,2), oma = c(0,0,2,0))
plot(model1_2022, pch = 16, sub.caption = "")
title(main="Diagnostics for model1_2022", outer=TRUE)

```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/d31503c8-8c40-43e4-9e93-888e4a3e4439)
As it can be seen, point 32 is an influential point and violates the LR condition; therefore, we need to exclude it.

## Excluding point 32 which is an influential point:
```{r}
data_2022 = data_2022%>% slice(-32)
```
<a name="model_2022_3"></a>
# 6.3.1 Backward elimination:
**We do feature selection using the Backward elimination method, which helps in identifying the most significant predictors and simplifying the model. Therefore we will end up with a model that is easier to interpret and potentially more robust.**

At this step day of the week is excluded since pretty much all of them are not significant(very high p-value:
```{r}

model2_2022 <- lm( NO2 ~ retail_and_recreation_percent_change_from_baseline+grocery_and_pharmacy_percent_change_from_baseline+parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+workplaces_percent_change_from_baseline+residential_percent_change_from_baseline  , data = data_2022)

# Summary
summary(model2_2022)

```
<img width="581" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/c6201035-30ad-43f8-9281-c4b1b4a6bcdb">

At this step residential_percent_change_from_baseline is excluded since has a very high p-value:

```{r}

model3_2022 <- lm( NO2 ~ retail_and_recreation_percent_change_from_baseline+grocery_and_pharmacy_percent_change_from_baseline+parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline+workplaces_percent_change_from_baseline  , data = data_2022)

# Summary
summary(model3_2022)

```
<img width="584" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/49338004-f409-44f3-98f7-e4875bcdcd3a">

### Checking the model using the diagnostic plot
```{r}
par(mfrow = c(2,2), oma = c(0,0,2,0))
plot(model3_2022, pch = 16, sub.caption = "")
title(main="Diagnostics for model3_2022", outer=TRUE)

```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/e5b9e2f0-8321-456e-821d-b1db97671a7c)


<a name="model_2022_4"></a>
# 6.3.4 Final Model 2022:

At this point workplaces_percent_change_from_baseline is excluded since it is not significant in the model:
```{r}

model4_2022 <- lm( NO2 ~ retail_and_recreation_percent_change_from_baseline+grocery_and_pharmacy_percent_change_from_baseline+parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline , data = data_2022)

# Summary
summary(model4_2022)

```
<img width="594" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/b0c0a45e-d5f2-4a4a-adb1-e3d525d3dd40">

Checking the diagnostic plots:

```{r}
par(mfrow = c(2,2), oma = c(0,0,2,0))
plot(model4_2022, pch = 16, sub.caption = "")
title(main="Diagnostics for m4", outer=TRUE)

```
![image](https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/8873d721-c17c-451a-a8a9-13bffeeb1713)


### The model seems good and all the following variables are significant.

1.parks_percent_change_from_baseline
2.transit_stations_percent_change_from_baseline
3.retail_and_recreation_percent_change_from_baseline

Now, we can consider model 4's result as our final model for 2022. We have good evidence that all the remaining variables impact the model, as the p values are small. Also, we met all the linear model conditions along the way.

#### NO2 = -15.30 + 1.33 × (retail_and_recreation_percent_change_from_baseline) + 0.10 × (parks_percent_change_from_baseline) - 1.36 × (transit_stations_percent_change_from_baseline)

## Interpratation of MLR Model4:

**Retail and Recreation** Each 1% increase from the baseline in retail and recreation activities is associated with an increase of approximately 1.33365 units in NO2 levels while we are controlling for parks_percent_change_from_baseline and retail_and_recreation_percent_change_from_baseline.


**Parks:** Each 1% increase in park visits from the baseline is associated with an increase of approximately 0.09555 units in NO2 levels controlling for retail_and_recreation_percent_change_from_baseline and retail_and_recreation_percent_change_from_baseline.

**Transit Stations:** Each 1% decrease from the baseline in transit station traffic is associated with a decrease of approximately 1.35743 units in NO2 levels,controlling for retail_and_recreation_percent_change_from_baseline and parks_percent_change_from_baseline.


## Overall F-test for model4:

• Full Model:NO2=β0 +β1×(parks_percent_change_from_baseline)+β2×(transit_stations_percent_change_from_baseline)+β3×(retail_and_recreation_percent_change_from_baseline)

#### H0 : No explanatory variables should be included in the model: β1 = β2 = · · · = βK = 0.***

#### HA : At least one explanatory variable should be included in the model: Not all βk’s = 0 for

The F-statistic from your model output has a very low p-value (2.092e-08), which provides strong evidence against the null hypothesis. This confirms that at least one of the β coefficients significantly differs from 0, indicating the necessity of including these explanatory variables in the model to explain the variability in NO2 levels.

Multiple R-squared: 0.787, implying that approximately 78.7% of the variability in NO2 levels is explained by the variations in retail and recreation, parks, and transit stations.

Adjusted R-squared: 0.7542, which takes into account the number of predictors in the model and provides a more precise measure of the model's explanatory power, adjusting for the degrees of freedom.

<a name="model_2022_5"></a>
# 6.3.5 2022_K-fold Cross-validation: 

### At this point K-fold cross-validation is performed to make sure that the model is not overfitting by comparing the results:

```{r}
control <- trainControl(method = "cv", number = 4)

# Training the model with linear regression
model_kfold_2022 <- train(NO2 ~ retail_and_recreation_percent_change_from_baseline+grocery_and_pharmacy_percent_change_from_baseline
               +parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline, data = data_2022, method = "lm", trControl = control)

# Print the results, including RMSE
print(model_kfold_2022)
```
<img width="419" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/67487be8-efa3-437d-acb9-7bdcb7aafaa9">

<a name="model_2022_6"></a>
# 6.3.3 2022_Random Forest:
### Let's perform random forest to make sure that our linear model is performing well enough.

```{r}
# Creating indices for the train set
trainIndex <- createDataPartition(data_2022$NO2, p = 0.75, list = FALSE, times = 1)

trainData <- data_2022[trainIndex, ]
testData <- data_2022[-trainIndex, ]

library(randomForest)
# Create the random forest model
rf_model_2022 <- randomForest(NO2 ~ retail_and_recreation_percent_change_from_baseline+grocery_and_pharmacy_percent_change_from_baseline
               +parks_percent_change_from_baseline+transit_stations_percent_change_from_baseline, data = trainData, ntree = 100, mtry = 3, importance = TRUE)

print(rf_model_2022)
```
<img width="562" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/7fa127d2-7d20-49b0-840d-4288c463581b">


```{r}
# Predict using the random forest model
predictions <- predict(rf_model_2022, testData)

mse <- mean((predictions - testData$NO2)^2)
rmse <- sqrt(mse)
mae <- mean(abs(testData$NO2 - predictions))
print(paste("Mean Absolute Error (MAE):", mae))
print(paste("MSE:", mse))
print(paste("RMSE:", rmse))

```
<img width="395" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/000ca961-6c48-4d19-8aa0-dd3131859973">



<a name="limit"></a>
# 8. Limitations:
There are some limitations of this project, which involves using Google Mobility:

1. **Data Representativeness and Coverage**: Google mobility data might not represent the entire population accurately. It only includes users who have location services enabled, potentially introducing a selection bias. The data may also vary in quality and completeness based on geographical regions and user demographics.

2. **Temporal and Spatial Variability**: The granularity of Google mobility data can vary, and it might not capture fine-scale temporal and spatial variability accurately. This limitation can affect the precision of MLR estimates and the understanding of NO2 level variations across different times and locations.

3. **Missing Pre-Lockdown Data**:Based on the following visualization, we observe that the trend for NO2 levels differs when comparing the data from 2020 and 2019. However, the absence of mobility data from before the COVID-19 pandemic limits our analysis. Without pre-lockdown mobility information, we cannot establish a clear baseline for comparing pre- and post-lockdown conditions. This gap makes it challenging to accurately quantify the full impact of lockdown measures on mobility patterns and NO2 levels.

   <img width="600" alt="image" src="https://github.com/Sheidahbb/Association-between-NO2-and-Human-Mobility/assets/113566650/5621d14b-d466-4913-82db-0b6b878bf3ab">

8.**Missing Google Mobility Data for Many Locations:** Google Mobility data is not captured for all locations in the United States. This incomplete coverage can lead to gaps in the dataset, affecting the comprehensiveness and accuracy of the analysis. Areas without mobility data may not be represented in the study, potentially skewing the results and limiting the generalizability of the findings. 

5. **Technological and Usage Biases**: Differences in smartphone usage, the version of Android installed, and the specific Google services utilized by users can affect the availability and accuracy of the mobility data. Additionally, variations in cell signal strength can influence data recording rates.

6. **Exposure Misclassification**: Traditional methods for estimating air pollution exposure often fail to account for human mobility and the time people spend in different environments. This misclassification can lead to inaccuracies in assessing the true exposure to NO2, especially if people spend significant time away from their primary residence.

7. **Interaction with Other Factors**: NO2 levels are influenced by various factors beyond human mobility, such as weather conditions, industrial activities, and traffic patterns. Not including these variables in your model could lead to omitted variable bias, affecting the reliability of your results.

