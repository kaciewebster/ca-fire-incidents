# ca-fire-incidents
A peak into the California Wildfires.

# Introduction
California is known to have some of the most devastating wildfires in the country, and in recent years, it feels as though they are getting worse. However, what does 'worse' mean? Are the number of fires increasing? Are the fires causing more damage to the land? Do people believe fires are a major incident? This presentation will focus on these questions and see how and if the wildfire seasons are changing.

# Data
The data used in this presentation were taken from [Kaggle](https://www.kaggle.com/ananthu017/california-wildfire-incidents-20132020). This data only focus on the years from 2013 to 2019. It lists each fire with information of where it was, what it took to extinguish it and the damage it caused. The data originally had 1,636 observations and 40 attributes each. However, this presentation will not be utilizing all of them. Let's take a look at the data.

# Libraries Used
<details>
  <summary></summary>
  
  ```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import folium
import folium.plugins as plugins
import scipy.stats as stats
from datetime import datetime, timedelta
from scipy.stats import pearsonr
  ```
</details>

# Load Data
<details>
  <summary></summary>
  
  ```python
full_fires_df = pd.read_csv('/Users/kaciewebster/Documents/dsi/ca-fire-incidents/California_Fire_Incidents.csv')
  ```
</details>

# Data Cleaning
To answer the questions above, only 10 columns are going to be utilized from this dataset. After the data cleaning, the dataset consists of 1,410 observations and 10 attributes.

1. UniqueID: an ID number for each fire

2. Name: the name of the fire

3. Counties: the county the fire started in

4. StartYear: the start year of the fire

5. StartMonth: the start month of the fire

6. StartDate: the start date of the fire

7. AcresBurned: number of acres damaged

8. Latitude: latitude of fire location

9. Longitude: longitude of fire location

10. MajorIncident: a binary response of True or False determining whether the fire was considered a 'major incident'
<details>
  <summary></summary>
  
  ```python
ca_fires_df = ca_fires_df.dropna(axis=0, subset=['AcresBurned'])
  # drops observations with null values in 'AcresBurned'.
  
ca_fires_df = ca_fires_df[ca_fires_df['AcresBurned'] != 0]
  # drops observations with 0 values in 'AcresBurned'.
  
ca_fires_df = ca_fires_df[(ca_fires_df['Latitude'] >= 32) & (ca_fires_df['Latitude'] <= 42) & (ca_fires_df['Longitude'] <= -114) & (ca_fires_df['Longitude'] >= -126)]
  # drops observations that reside far outside of California.
  
ca_fires_df[(ca_fires_df['Latitude'] >= 38) & (ca_fires_df['Latitude'] <= 41) & (ca_fires_df['Longitude'] <= -115) & (ca_fires_df['Longitude'] >= -118)]
ca_fires_df[ca_fires_df['Counties'] == 'State of Nevada']
  # locates which values are from Nevada.
  
ca_fires_df = ca_fires_df[ca_fires_df['Counties'] != 'State of Nevada']
ca_fires_df = ca_fires_df[ca_fires_df['Name'] != 'Tram Fire']
  # drops observations from Nevada.
  
ca_fires_df = ca_fires_df[ca_fires_df['StartYear'] >= 2013]
  # drops observations that are not in 2013-2019.
  
ca_fires_df = ca_fires_df.groupby('UniqueId').max().reset_index()
  # groups fires by their ID to avoid double counting bigger fires.
  ```
</details>

# EDA
<details>
  <summary>Line Plots</summary>
  
  ```python
num_fires = ca_fires_df.groupby('StartYear')['AcresBurned'].count()
years = sorted(ca_fires_df['StartYear'].unique())
    
fig, ax = plt.subplots()
ax.plot(years, num_fires, color='red')
ax.set_title('Number of Fires per Year')
ax.set_xlabel('Year')
ax.set_ylabel('Fires')
fig.savefig('num-fires-line-plot.png')
# creates 'Number of Fires' plot.

sum_acres = ca_fires_df.groupby('StartYear')['AcresBurned'].sum()
years = sorted(ca_fires_df['StartYear'].unique())

fig, ax = plt.subplots()
ax.plot(years, sum_acres, color='red')
ax.set_title('Total Acres Burned per Year')
ax.set_xlabel('Year')
ax.set_ylabel('Acres')
fig.savefig('total-acres-line-plot.png')
# creates 'Total Acres' plot.

median_acres = ca_fires_df.groupby('StartYear')['AcresBurned'].median()
years = sorted(ca_fires_df['StartYear'].unique())

fig, ax = plt.subplots()
ax.plot(years, median_acres, color='red')
ax.set_title('Median Acres Burned per Year')
ax.set_xlabel('Year')
ax.set_ylabel('Acres')
fig.savefig('med-acres-line-plot.png')
# creates 'Median Acres' plot.

max_acres = ca_fires_df.groupby('StartYear')['AcresBurned'].max()
years = sorted(ca_fires_df['StartYear'].unique())

fig, ax = plt.subplots()
ax.plot(years, max_acres, color='red')
ax.set_title('Acres of Largest Fire per Year')
ax.set_xlabel('Year')
ax.set_ylabel('Acres')
fig.savefig('max-acres-line-plot.png')
# creates 'Max Acres' plot.
    
plt.show()
  ```
</details>

![](img/num-fires-line-plot.png) ![](img/total-acres-line-plot.png)

![](img/med-acres-line-plot.png) ![](img/max-acres-line-plot.png)

These plots...

<details>
  <summary>Bar Plots</summary>
  
  ```python
def parse_grouped_sums(grouped_df, year_lst):
    '''
    Creates a list of dataframes parsed by year.
    Parameters: grouped dataframe, list of years
    Returns: list of dataframes
    '''
    parse_lst = []
    
    for year in year_lst:
        parse_lst.append(grouped_df[grouped_df['StartYear']==year])
    return parse_lst
    
year_lst = list(range(2013, 2020))
df_lst = parse_grouped_sums(grouped_acres_sum, year_lst)
  # creates a list where each element is a dataframe containing the distribution of AcresBurned per month.

month_labels = ['Jan.', 'Feb.', 'Mar.', 'Apr.', 'May', 'Jun.', 'Jul.', 'Aug.', 'Sep.', 'Oct.', 'Nov.', 'Dec.']
fig, axs = plt.subplots(7, 1, figsize=(8, 25))

for idx, ax in enumerate(axs.flatten()):
    ax.bar(df_lst[idx]['StartMonth'], height=df_lst[idx]['AcresBurned'], color='red')
    # creates a bar plot for each year.
        
    ax.set_xticks(list(range(1, 13)))
    ax.set_xticklabels(month_labels, rotation=45)
        
    ax.set_title(f'Acres Burned per Month ({year_lst[idx]})')
    ax.set_xlabel('Months')
    ax.set_ylabel('Acres')
    fig.savefig('months-bar-plots.png')
    
fig.tight_layout()
plt.show()
  ```
</details>

![](img/months-bar-plots.png)

These plots ...

# Hypothesis Testing
1. Correlation Test

H0: There is no trend for the number of acres burned by the California Fires over the years 2013-2019.

H1: The number of acres burned by the California Fires has been increasing over the years.

<details>
  <summary>Scatter Plot</summary>
  
  ```python
fig, ax = plt.subplots(figsize=(16, 4))
ax.scatter(ca_fires_df['StartDate'], ca_fires_df['AcresBurned'])

ax.set_title('Acres Burned vs. Time')
ax.set_xlabel('Time')
ax.set_ylabel('Acres Burned')
ax.set_ylim(10**3, 10**6)

plt.show()
fig.savefig('time-vs-acres-scatter.png')
  ```
</details>

![](img/time-vs-acres-scatter.png)

<details>
  <summary>Correlation Coefficient and P-Value</summary>
  
  ```python
ca_fires_copy = ca_fires_df.copy()
ca_fires_copy = ca_fires_copy.sort_values('StartDate')
# creates a copy and sorts the copy in chronological order.

time_diff = []
date_lst = list(ca_fires_copy['StartDate'].sort_values())
for idx in range(0, len(date_lst)-1):
    time_diff.append(date_lst[idx+1]-date_lst[0])
# creates a list of time differences from the data point to the first recorded observation.

time_diff.insert(0, timedelta(days=0))
# inserts a 0 since the first day is the initial day.

ca_fires_copy['TimeDifference'] = time_diff
# creates a new column with those calculated differences.

time_diff_int = []
for date in list(ca_fires_copy['TimeDifference']):
    time_diff_int.append(date.days)
# creates a list that changes the timedelta data to integers.

ca_fires_copy['TimeDiffAsInt'] = time_diff_int
# adds the integer list as a new column.

corr, p_value = stats.pearsonr(ca_fires_copy['TimeDiffAsInt'], ca_fires_copy['AcresBurned'])
corr, p_value
# executes the correlation test and returns a coefficient and p-value.
  ```
</details>
