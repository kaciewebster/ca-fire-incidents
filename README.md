# ca-fire-incidents
A peak into the California Wildfires.

# Introduction
California is known to have some of the most devastating wildfires in the country, and in recent years, it feels as though they are getting worse. However, what does 'worse' mean? Are the number of fires increasing? Are the fires causing more damage to the land? Do people believe fires are a major incident? This presentation will focus on these questions and see how and if the wildfire seasons are changing.

# Data
The data used in this presentation were taken from [Kaggle](https://www.kaggle.com/ananthu017/california-wildfire-incidents-20132020). This data only focus on the years from 2013 to 2019. It lists each fire with information of where it was, what it took to extinguish it and the damage it caused. The data originally had 1,636 observations and 40 attributes each. However, this presentation will not be utilizing all of them. Let's take a look at the data.

# Libraries Used
<details>
  <summary></summary>
  <pre><code>
    import numpy as np
    import pandas as pd
    import matplotlib.pyplot as plt
    import folium
    import folium.plugins as plugins
    import scipy.stats as stats
    from datetime import datetime, timedelta
    from scipy.stats import pearsonr
  </pre></code>
</details>

# Load Data
<details>
  <summary></summary>
  <pre><code>
  full_fires_df = pd.read_csv('/Users/kaciewebster/Documents/dsi/ca-fire-incidents/California_Fire_Incidents.csv')
  </pre></code>
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
  <pre><code>
    ca_fires_df = ca_fires_df.dropna(axis=0, subset=['AcresBurned'])
    ca_fires_df = ca_fires_df[ca_fires_df['AcresBurned'] != 0]
    ca_fires_df = ca_fires_df[(ca_fires_df['Latitude'] >= 32) & (ca_fires_df['Latitude'] <= 42) & (ca_fires_df['Longitude'] <= -114) & (ca_fires_df['Longitude'] >= -126)]
    ca_fires_df = ca_fires_df[ca_fires_df['Counties'] != 'State of Nevada']
    ca_fires_df = ca_fires_df[ca_fires_df['Name'] != 'Tram Fire']
    ca_fires_df = ca_fires_df[ca_fires_df['StartYear'] >= 2013]
    ca_fires_df = ca_fires_df.groupby('UniqueId').max().reset_index()
  </pre><code>
</details>
