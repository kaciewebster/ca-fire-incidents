# California Fire Incidents
A peek into the California Wildfires.

<p align="center">
    <img src="img/85.jpeg" height="400" width="600"></p>

# Introduction
California is known to have some of the most devastating wildfires in the country, and in recent years, it feels as though they are getting worse. However, what does "worse" mean? According to the United States Forest Service, they "evaluate the SFDI (Severe Fire Danger Index) against the number of newly reported wildfires and total area burned from agency fire reports". Therefore, I will analyze the change in number of fires over the years against the change in the nature of fires to see if the wildfires are getting worse.

# Data
The data used in this project was taken from [Kaggle](https://www.kaggle.com/ananthu017/california-wildfire-incidents-20132020). This data only focused on the years from 2013 to 2019. It lists each fire with information of where it was, what it took to extinguish it and the damage it caused. The data originally had 1,636 observations and 40 attributes each. However, this project will not be utilizing all of them.

To answer the question above, only 10 columns are going to be utilized from this dataset. After the data cleaning, the dataset consists of 1,410 observations and 10 attributes.

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

# Number of Fires vs. Nature of Fires

To see how these plots and calculations were produced or found, please reference the [ca-fires-final.ipynb](/src/ca-fires-final.ipynb) file in the src folder.

To begin my analysis on the change in the number and nature of these fires, I wanted to see where these fires a located. Here is a map showing the locations of each fire for each year. You can see that the frequency of fires increases over the years, and they tend to cluster around Southern California, the bay area, and Sacramento.

<p align="center">
    <img src="img/num-fires.gif" height="400" width="600"></p>

As a sanity check to make sure that the map is showing an increase in fires over the years, here are the numbers to prove it. You can see that the number of fires are increasing with a spike in 2017 and a decrease after that.

| Year  | Number of Fires |
| :--:  | :--:  |
| 2013  | 136 |
| 2014  | 69  |
| 2015  | 91  |
| 2016  | 149 |
| 2017  | 411 |
| 2018  | 297 |
| 2019  | 257 |

However, the chart above only represents the number of fires. Some of these fires could be little spots that were able to be stomped out. Let's get a better look at the nature of these fires or the amount of damage they caused using acres burned as the metric. Here is a line graph of the total acres burned, maximum acres burned, and median acres burned. The log was taken for each of these data points in order to find the general trend of this data. You can see that the total acres burned and maximum acres burned generally increase until 2019. However, the median acres burned peaked in 2014 and decreased after that. This indicates that recent years have seen a few large fires, but 2014 had the most consistently large fires compared to other years.

<p align="center">
    <img src="img/acres-burned-line-plot.png" height="400" width="600"></p>

# Hypothesis Testing

1. Correlation Test

H0: There is no trend for the number of acres burned by the California Fires over the years 2013-2019.

H1: The number of acres burned by the California Fires has been increasing over the years.

![](img/time-vs-acres-scatter.png)

The level of significance used is 0.05 or 5%. Since the p-value, 0.6373, is greater than our level of significance, the data fails to reject the null hypothesis. Therefore, there is no evidence that the number of acres burned are increasing over the years 2013-2019.

2. Bayes Hypothesis Test

![](img/posterior-plots.png)

These distribution plots show that the probability of a fire being considered a major event decreases from 2013-2019. After calculations, the probability of a fire being considered a major increase in 2013 less often than in 2019 was 0%. This shows that there was definitely a decrease in people's perception of whether a fire was a major incident. In comparison to the year 2018, the probability was still only 6%. People do not believe fires are major incidents as much as they did in 2013.

# Conclusion
Overall, the number of fires in California are increasing along with when these fires are occurring. Wildfire season is changing from spring and summer to summer and fall. However, the 'severity' or how much damage these fires are causing cannot be proven to be increasing. People are also not as concern for fires as a major incident as they were in the past. This could be due to many reasons. The number of fires are increasing, but the acres do not appear to be increasing. So, people are probably only seeing smaller fires, which is not as major as a bigger fire.

# Further Testing
Some things to consider for further testing would be:
1. To increase the time period: this would give a better idea of the progression of wildfire seasons in California
2. Test for a relationship with temperature: temperature could shed some light on how wildfires are changing along with climate change


