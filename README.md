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

To take a look at where the fires are distributed across California refer to the time-map.html in the img folder.

<details>
  <summary></summary>
  
  ```python
df_copy = ca_fires_df.copy()
df_years_list = []
for year in df_copy['StartYear'].sort_values().unique():
    df_years_list.append(df_copy.loc[df_copy['StartYear']==year, ['Latitude', 'Longitude']].groupby(['Latitude', 'Longitude']).sum().reset_index().values.tolist())
# creates a list of lists where each element is a year and each element in that list element contains the latitudes and longitudes of each fire.
    
m = folium.Map(location=[34.0522, -118.2437], zoom_start=5)
plugins.HeatMapWithTime(df_years_list, index=[2013, 2014, 2015, 2016, 2017, 2018, 2019], radius=5, min_opacity=0.5, max_opacity=0.8, use_local_extrema=True).add_to(m)
  ```
</details>

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

These line plots help to reveal some insight into what kind of fires are burning throughout California. The line plot describing the number of fires shows that despite the decrease in fires after 2017, the number of fires is growing overall. However, does that mean that there is more damage? The other three line plots describe the nature of these fires. 'Total AcresBurned per Year' shows that the total area of land affected by these fires was increasing until 2019. The steep decrease implies that 2019 was a unique year for wildfires. 'Median AcresBurned per Year" shows that the most years had mostly small fires with the exception of 2014. The median fires for all the years other than 2014 were relatively small compared to the median fire for 2014. 'Acres of the Largest Fire per Year' helps to show where the outliers reside. Since 2018 has the maximum acres burned for all the years, this indicates that there was a terrible fire that occurred. The size of that fire was about 1.5 times bigger than the largest fire of 2013.

<details>
  <summary>Bar Plots</summary>
  
  ```python
grouped_acres_sum = ca_fires_df.groupby(['StartYear', 'StartMonth']).sum('AcresBurned')['AcresBurned'].reset_index()
# groups the dataset by StartYear and StartMonth and sums the AcresBurned.

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

These bars plot show that in the earlier years, more acres are burned in the spring and summer time. However, by 2019, there is a shift where more acres are burning in the summer and fall time.

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

Analysis ...

2. Bayes Hypothesis Test

<details>
  <summary>Posterior Distribution Plot</summary>
  
  ```python
mi_lst = []
for year in year_lst:
    mi_lst.append(ca_fires_df[ca_fires_df['StartYear']==year]['MajorIncident'])
# creates a list where each element is the data from the column, 'MajorIncident', of one year.

nk_dct = {}
for bool_lst in mi_lst:
    nk_dct[len(bool_lst)] = sum(bool_lst)
# creates a dictionary where the key is the number of observations in that year and the value is the number of 'MajorIncidents'.

post_dist = []
for n, k in nk_dct.items():
    post_dist.append(stats.distributions.beta(a = 1 + k, b = 1 + n - k))
# runs a posterior distribution for each year.

fig, ax = plt.subplots(figsize=(10, 6))
x = np.linspace(0,1,1000)
year_lst = [2013, 2014, 2015, 2016, 2017, 2018, 2019]
for idx, dist in enumerate(post_dist):
    ax.plot(x, dist.pdf(x), label=f'{year_lst[idx]}')
    ax.legend()
    ax.set_title('Posterior Probabilities of a Major Incident by Year')
    ax.set_xlabel('p')
    ax.set_ylabel('pdf')

fig.savefig('posterior-plots.png')
# plots the posterior distributions for each year.
  ```
</details>

![](img/posterior-plots.png)

<details>
  <summary>Calculations</summary>
  
  ```python
beta_2013 = post_dist[0]
beta_2019 = post_dist[-1]
# assigns the beta distributions of 2013 and 2019.

sim_2013 = beta_2013.rvs(size=10000)
sim_2019 = beta_2019.rvs(size=10000)
# creates 10,000 simulations from the 'MajorIncident' column.

(sim_2019 > sim_2013).mean()
# calculates what the probability is that P(MajorIncident|2019) > P(MajorIncident|2013).

beta_2013 = post_dist[0]
beta_2018 = post_dist[-2]
sim_2013 = beta_2013.rvs(size=10000)
sim_2018 = beta_2018.rvs(size=10000)
(sim_2018 > sim_2013).mean()
# does the same thing as above but for 2013 and 2018.
  ```
</details>

