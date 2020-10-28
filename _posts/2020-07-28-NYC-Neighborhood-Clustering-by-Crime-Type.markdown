---
layout: post
title:  "NYC Neighborhood Clustering by Crime Type"
date:   2020-07-28 00:16:45 -0400
categories: jekyll update
---

Documentation in progress..

Github: https://github.com/KhokiBernier/NYC-Neighborhood-Clustering/blob/main/NYC%20Neighborhood%20Clustering%20EDITED.ipynb

# **Project Overview**

Clustered New York City nieghborhoods by crime type per 1,000 residents and neighborhood population using K-Means algorithm in Python and visualized data in Tableau

<iframe frameborder="0" height="800" width="1400" scrolling="no" src="https://public.tableau.com/views/NYCClusteringEDITED/DashboardDark?:language=en&:display_count=y&publish=yes&:origin=viz_share_link:showVizHome=no&:embed=yes"> </iframe>

<!---
<img src="/assets/img/NYC-Neighborhood-Dashboard.png">
--->

# **Libraries used:**

- pandas
- numpy
- matplotlib
- seaborn
- sklearn
- plotly
- statistics

# **Links**

- NYC Arrest dataset: https://data.cityofnewyork.us/Public-Safety/NYPD-Arrest-Data-Year-to-Date-/uip8-fykc
- NYC Neighborhood by population file: https://data.cityofnewyork.us/City-Government/New-York-City-Population-By-Neighborhood-Tabulatio/swpk-hqdp
- NYC Neighborhood Coordinates: https://data.cityofnewyork.us/City-Government/Neighborhood-Tabulation-Areas-NTA-/cpf4-rkhq
- NYC Map to verify Midtown South: https://maps.nyc.gov/crime/
- US Crime Statistics: https://ucr.fbi.gov/crime-in-the-u.s/2012/crime-in-the-u.s.-2012/tables/29tabledatadecpdf

<!--- Code to get Neighborhoods and venues: https://codekarim.com/node/57 (Note: I do not take credit for this code)
<!--- Missing Neighborhood Populations: https://popfactfinder.planning.nyc.gov/
--->

# **Steps Taken**

<!---
1. Downloaded and cleaned file that contained 5 million+ rows of NYC crime occurances, descriptions, and coordinates
2. Pulled NYC neighborhood coordinates as well as coordinates from all venues in each neighborhood by following steps in this link https://codekarim.com/node/57 
3. Mapped crime location to neighborhood by finding minimum Euclidean distance between crime location and all venues
4. Mapped neighborhood population to nieghborhood using 'NYC Neighborhood by population file' dataset
  - Built several functions to map neighborhoods, as dataset nieghborhood syntax was not the same
5. Grouped, pivoted and normalized data and used K-Means Clustering algorithm to determine clusters based on the following variables (per 1,000 residents):
- Violent Crime
- Theft
- Drug Related Crime
- Traffic Crime
- General Non-Violent Crime (loittering, gambling, health code violations, etc)
- Neighborhood Population
6. Performed K-Means Clustering
7. Visualized data in Tableau
--->

1. Download and format data files
2. Feature Engineering
3. Outlier Analysis
4. Clustering
5. Create Tableau Viz

All code can be found on github (linked above)

**Step 1: Download and format data files**

```python
import pandas as pd
import statistics
import numpy as np

#dataframe with coordinates
df_coordinates = pd.read_csv('nynta.csv')
#dataframe with population counts
nyc_pop_data = pd.read_csv('NYC Population Data.csv')
#dataframe with crimes
df_crime = pd.read_csv('NYPD_Arrests_Data__Historic_.csv')

df_crime.ARREST_DATE = pd.to_datetime(df_crime.ARREST_DATE)
df_crime = df_crime[df_crime.ARREST_DATE.dt.year >= 2017]

df_crime.Latitude = df_crime.Latitude.astype(float)
df_crime.Longitude = df_crime.Longitude.astype(float)

df_crime.Latitude = df_crime.Latitude.replace([np.inf, -np.inf], np.nan)
df_crime.Longitude = df_crime.Longitude.replace([np.inf, -np.inf], np.nan)

df_crime.Latitude = df_crime.Latitude.dropna()
df_crime.Longitude = df_crime.Longitude.dropna()

df_crime.Latitude.isna().any()
```

<!---
**Step 1: Downloaded and cleaned file that contained 5 million+ rows of NYC crime occurances, descriptions, and coordinates**

- Dataset taken from this link: https://data.cityofnewyork.us/Public-Safety/NYPD-Arrest-Data-Year-to-Date-/uip8-fykc
- Clean and organize data

```python
import pandas as pd
import numpy as np

df_crime = pd.read_csv('NYPD_Arrests_Data__Historic_.csv')
#Convert arrest to date and remove data prior to 2017
df_crime.ARREST_DATE = pd.to_datetime(df_crime.ARREST_DATE)
df_crime = df_crime[df_crime.ARREST_DATE.dt.year >= 2017]
#Convert Latitude and Longitude to floats
df_crime.Latitude = df_crime.Latitude.astype(float)
df_crime.Longitude = df_crime.Longitude.astype(float)
#Replace any infinity values with nan and check if there are any nan values
df_crime.Latitude = df_crime.Latitude.replace([np.inf, -np.inf], np.nan)
df_crime.Longitude = df_crime.Longitude.replace([np.inf, -np.inf], np.nan)
df_crime.Latitude.isna().sum()
#drop nan values (there was only 1)
df_crime.Latitude = df_crime.Latitude.dropna()
df_crime.Longitude = df_crime.Longitude.dropna()
```

Look at the distinct offense description values and determine groupings:

```python
#listing them out this way makes them easy to copy paste
df_crime.OFNS_DESC.unique()
```

```python
violent = ['MURDER & NON-NEGL. MANSLAUGHTER','HOMICIDE-NEGLIGENT-VEHICLE', 'MURDER & NON-NEGL. MANSLAUGHTE',
           'HOMICIDE-NEGLIGENT,UNCLASSIFIED','HOMICIDE-NEGLIGENT-VEHICLE','HOMICIDE-NEGLIGENT,UNCLASSIFIE','RAPE',
           'FELONY ASSAULT', 'ASSAULT 3 & RELATED OFFENSES','JOSTLING''SEX CRIMES','FORCIBLE TOUCHING',
           'ROBBERY','OFFENSES RELATED TO CHILDREN','KIDNAPPING & RELATED OFFENSES','KIDNAPPING']
traffic = ['OTHER TRAFFIC INFRACTION','VEHICLE AND TRAFFIC LAWS','INTOXICATED & IMPAIRED DRIVING',
           'INTOXICATED/IMPAIRED DRIVING','UNAUTHORIZED USE OF A VEHICLE 3 (UUV)','PARKING OFFENSES',
           'UNAUTHORIZED USE OF A VEHICLE']
non_violent = ['THEFT-FRAUD','FORGERY','FRAUDS','OFFENSES INVOLVING FRAUD','FRAUDULENT ACCOSTING','ARSON', 'OTHER',                    'DANGEROUS WEAPONS','UNLAWFUL POSS. WEAP. ON SCHOOL GROUNDS','UNLAWFUL POSS. WEAP. ON SCHOOL','CRIMINAL                TRESPASS','HARRASSMENT 2','OFFENSES AGAINST PUBLIC SAFETY','HARASSMENT']
theft = ['GRAND LARCENY','PETIT LARCENY','OTHER OFFENSES RELATED TO THEF','THEFT OF SERVICES',
         'OTHER OFFENSES RELATED TO THEFT', 'PETIT LARCENY','GRAND LARCENY OF MOTOR VEHICLE',
         'POSSESSION OF STOLEN PROPERTY','POSSESSION OF STOLEN PROPERTY 5','BURGLARY']
drugs = ['DANGEROUS DRUGS','LOITERING FOR DRUG PURPOSES']

crime_group_list = []

for i in df_crime['OFNS_DESC']:
    if (i in violent) == True:
        crime_group_list.append('Violent')
    elif (i in non_violent) == True:
        crime_group_list.append('Non Violent')
    elif (i in traffic) == True:
        crime_group_list.append('Traffic')
    elif (i in theft) == True:
        crime_group_list.append('Theft')
    elif (i in drugs) == True:
        crime_group_list.append('Drug')
    else:
        crime_group_list.append('Other')
        
df_crime['Crime Group'] = crime_group_list
```
--->

**Step 2: Feature Engineering**

The following features were added:

- Crime Grouping
- Neighborhood Population
- Crimes Per 1,000 Population
- Neighborhood Center Coordinates (Note: This feature is for Tableau Visualization)
- Neighborhood Boarder Coordinates (Note: This feature is for Tableau Visualization)

**Step 3. Outlier Analysis**

Because the K-Means algorithm uses the mean of clusters, it’s sensitive to outliers and does not perform well with them. Additionally, I’d like to have more meaningful clusters, and our outlier data seems to be of more or less the same variety - very high crime neighborhoods. I decided to group these outliers into 1 ‘High Crime’ cluster, then create a dataset that excludes these neighborhoods and re-run the K-Means algorithm. Outliers accounted for 5 out of 193 (~2.5%) of our neighborhood population.

**Step 4. Clustering**

Clustering is a form of unsupervised exploratory analysis that aims to segment data based on similar traits. In our case, we a used k-means clustering algorithm, where the number of clusters (K) is specified prior to clustering. In K-Means clustering, k (the selected number of clusters) centroids are chosen as random data points. Our data points are then taken and assigned to the nearest centroid based on Euclidean distance. Once all data points are assigned to a centroid, the mean of each cluster is calculated and determined to be the new centroids. All data points are then reassigned to the nearest new centroid value. This process continues until the data points stop changing clusters, and once this occurs the total variation amongst points in the clusters is then calculated. This entire process is repeated x amount of times and the sequence with the lowest variation is used to cluster the data.

The clusters are described as follows:

- Low Crime: Low Population (69 Neighborhoods)
- Low Crime: High Population (33 Neighborhoods)
- Medium Crime: Mid Population (59 Neighborhoods)
- High Violent/Theft, Medium Traffic: Mid Population (25 Neighborhoods)
- High Crime: Low Population (2 Neighborhoods)
- Extremely High Crime: Extremely Low Population (4 Neighborhoods) 

Here we have some clusters around degrees of crime and population, and some insight into neighborhoods with high violence and theft with mid level traffic crime. However, it does appear that generally when 1 type of crime is high in a neighborhood all types of crime are high as well. While that in itself is a bit of a relevation, particuralry with types of crime such as traffic, if the data set variables were less linear K-Means clustering could output more interesting clusters.

**Step 5. Create Tableau Viz**

<iframe frameborder="0" height="800" width="1400" scrolling="no" src="https://public.tableau.com/views/NYCClusteringEDITED/DashboardDark?:language=en&:display_count=y&publish=yes&:origin=viz_share_link:showVizHome=no&:embed=yes"> </iframe>

Here's another dashboard I created with just the crime data.

<img src="/assets/img/NYC-Crime-Data.png">

[NYC Crime Dashboard Link](https://public.tableau.com/profile/kb8769#!/vizhome/NYCCrimeCoordinates/Dashboard1)










<!---

COMMENTED OUT STARTS HERE

**Crime Grouping**

Each crime has a description, so we'll group them into categories.

```python
#Listing the descriptions out this way makes them easy to copy and paste
df_crime.OFNS_DESC.unique()
```
Here we create our 5 groupings (Violent, Theft, Drugs, Traffic, and Non_Violent_Other)

```python
violent = ['MURDER & NON-NEGL. MANSLAUGHTER','HOMICIDE-NEGLIGENT-VEHICLE', 'MURDER & NON-NEGL. MANSLAUGHTE',
    'HOMICIDE-NEGLIGENT,UNCLASSIFIED','HOMICIDE-NEGLIGENT-VEHICLE','HOMICIDE-NEGLIGENT,UNCLASSIFIE', 
    'FELONY ASSAULT', 'ASSAULT 3 & RELATED OFFENSES','JOSTLING', 'RAPE','SEX CRIMES','FORCIBLE TOUCHING',
    'ROBBERY','OFFENSES RELATED TO CHILDREN','KIDNAPPING & RELATED OFFENSES','KIDNAPPING']

traffic = ['OTHER TRAFFIC INFRACTION','VEHICLE AND TRAFFIC LAWS','INTOXICATED & IMPAIRED DRIVING',
           'INTOXICATED/IMPAIRED DRIVING','UNAUTHORIZED USE OF A VEHICLE 3 (UUV)','PARKING OFFENSES',
           'UNAUTHORIZED USE OF A VEHICLE']

non_violent = ['THEFT-FRAUD','FORGERY','FRAUDS','OFFENSES INVOLVING FRAUD','FRAUDULENT ACCOSTING','ARSON',
               'CRIMINAL TRESPASS','DANGEROUS WEAPONS','UNLAWFUL POSS. WEAP. ON SCHOOL GROUNDS',
               'UNLAWFUL POSS. WEAP. ON SCHOOL','Tresspassing','HARRASSMENT 2','OFFENSES AGAINST PUBLIC SAFETY',
               'HARASSMENT','PROSTITUTION & RELATED OFFENSES']

theft = ['GRAND LARCENY','PETIT LARCENY','OTHER OFFENSES RELATED TO THEF','THEFT OF SERVICES',
         'OTHER OFFENSES RELATED TO THEFT', 'PETIT LARCENY','GRAND LARCENY OF MOTOR VEHICLE',
         'POSSESSION OF STOLEN PROPERTY','POSSESSION OF STOLEN PROPERTY 5','BURGLARY']

drugs = ['DANGEROUS DRUGS','LOITERING FOR DRUG PURPOSES']

crime_group = []

for i in df_crime.OFNS_DESC:
    if (i in violent) == True:
        crime_group.append('Violent')
    elif (i in non_violent) == True:
        crime_group.append('Non Violent')
    elif (i in traffic) == True:
        crime_group.append('Traffic')
    elif (i in theft) == True:
        crime_group.append('Theft')
    elif (i in drugs) == True:
        crime_group.append('Drug')
    else:
        crime_group.append('Non Violent')
        
df_crime['Crime Group'] = crime_group
```

**Neighborhood Population**

Map NYC Population to Neighborhoods

```python

```
--->

<!---
**Step 2: Pulled NYC neighborhood coordinates as well as coordinates from all venues in each neighborhood by following steps in this link https://codekarim.com/node/57**

- I copied the steps listed in the link and adjusted a few things in order to get all neighborhood and venue coordinates for all 5 boroughs
- The code was created by 2 guys named Alex Aklson and Polong Lin, and involves working with JSON files

**Step 3: Mapped crime location to neighborhood by finding minimum Euclidean distance between crime location and all venues**

- Now that we have crime coordinates and neighborhood venue coordinates, we can map each crime to a neighborhood
- Euclidean distance represents the distance between two points, and can be used for n dimensions (our example has 2)
- Formula: d(x,y) = sqrt((x1-x2)^2 + (y1-y2)^2)

First we put our venue latitudes and longitudes into a list of coordinates
```python
coordinates = []
for lat, long in df_ny[['Venue Latitude','Venue Longitude']].itertuples(index=False):
    coordinates.append((lat,long))
```
Next, we define 2 functions. 1 to calculate Euclidean distance and 1 to find the closest pair of coordinates out of a list of coordinates.

```python
from math import sqrt

def Euclidean_Distance(p, q):
    return sqrt(((p[0] - q[0]) ** 2) + ((p[1] - q[1]) ** 2))

def closest(cur_pos, positions):
    low_dist = float('inf')
    closest_pos = None
    for pos in positions:
        dist = Euclidean_Distance(cur_pos,pos)
        if dist < low_dist:
            low_dist = dist
            closest_pos = pos
    return closest_pos
```

We can now loop through


```python
from datetime import datetime

neighborhood_list = []
tracker = 0

for lat,long in df_crime[['Latitude','Longitude']].itertuples(index=False):
    point = (lat,long)
    closest_point = closest(point, coordinates)
    #setting variable equal to nearest venue based on closest point coordinates
    ind = df_ny[(df_ny['Venue Latitude'] == closest_point[0]) & (df_ny['Venue Longitude'] == closest_point[1])].Neighborhood.values[0]
    neighborhood_list.append(ind)
    tracker+=1
    #This helps us track our script, it takes about 10 minutes for every 100,000 rows
    if tracker % 100000 == 0:
        print(tracker)
        print(datetime.now(tz=None))
df_crime['Neighborhood'] = neighborhood_list
```


**Step 4. Mapped neighborhood population to nieghborhood using 'NYC Neighborhood by population file' dataset**

- First, import the dataset into a data frame (link to dataset: https://data.cityofnewyork.us/City-Government/New-York-City-Population-By-Neighborhood-Tabulatio/swpk-hqdp)

```python
nyc_pop_data = pd.read_csv('NYC Population Data.csv')
#The dataset has population data for 2000 and 2010, we'll look at 2010 since it's the most recent and NYC population hasn't changed too much since then
nyc_pop_data = nyc_pop_data[nyc_pop_data.Year == 2010]
```

- I noticed that the counts of neighborhoods between this dataset and our crime mapped dataframe were different, so I performed the following to investigate the discrepency

```python
df_crime.Neighborhood.nunique() # this returned 301, so we have 301 neighborhoods that we need population counts for

ctr_y = 0
ctr_n = 0

not_in_neighborhoods = []

for i in df_crime['Neighborhood'].unique():
    if (i in nyc_pop_data['NTA Name'].values) == True:
        ctr_y+=1
    else:
        ctr_n+=1
        not_in_neighborhoods.append(i)
        
print('In both = ',ctr_y)
print('not in both = ',ctr_n)
```

- In both = 86
- not in both = 215

We only have about a quarter of the neighborhoods mapped, not very good.
To investigate, I ran the same loop above but switched the dataframes in order to see the neighborhoods in the NYC Population dataset that did not get mapped to a df_crime dataset.

<img src="/assets/img/hyponated-neighborhoods.png">

Looking at this output, we immediately notice the hyphons. Non of our df_crime neighborhoods have hyphons, but instead each hyphonated item is listed as its own neighborhood. For example:

Crime dataframe:
- Claremont
- Bathgate

Population dataframe:
- Claremont-Bathgate

Because our population counts are dependent on the Population dataset, our crime dataset will have to abide by the population dataset 'Rules' in terms of heirarchy. So my solution was to create a dictionary with the keys as the non-hyphonated neighborhoods and place the hyphonated neighborhoods as the value, resulting in the below:

- {'Claremont' : 'Claremont-Bathgate',
- 'Bathgate'. : 'Claremont-Bathgate',}

This was achieved using the loop below:

```python
key_list = []
value_list = []
ctr = 0
for i in not_in_neighborhoods:
    if '-' in i:
        #split function creates a list, we're using '-' as a seperator
        key2 = i.split('-')
        for n, val in enumerate(key2):
            #appending each element of our list into our keys
            key_list.append(val.replace("'", ""))
            #appending the original hyphonated value into our list of values
            value_list.append(i.replace("'", ""))
#create our dictionary
neighborhood_dictionary = dict(zip(key_list, value_list))
#replace our crime neighborhoods using dictionary
df_crime['Neighborhood'] = df_crime['Neighborhood'].replace(neighborhood_dictionary)
```

Re-running our loop from above to determine how many neighborhoods we have mapped resulted in:

- In both =  152
- not in both =  80

The total number of neighborhoods decreased because of our hyphonations, so we went from:

- Before: 215 missing
- After: 80 missing

After rechecking the neighborhoods in the population file that did not get mapped, I noticed many of our population neighborhoods specify 'East', 'West', 'North', 'South', and our crime data does not. For example:

Population file:
- Sunset Park West
- Sunset Park East

Crime file:
- Sunset Park

In this case, because we can't specify which Sunset Park (East or West) the crime occured, I grouped Sunset Park East and Sunset Park West in the population file to be Sunset Park, and combined the population numbers. This eliminated another 10 neighborhoods. 

For the remaining 70, they seemed to not be in the file, so I mapped them manually using this website https://popfactfinder.planning.nyc.gov/. This was definetly the most boring part of the project, but it only took about an hour and it requires very little critical thinking so you can listen to music while doing it.

We now have our neighborhoods in a standardized format between our population and crime dataframes.

**Step 5. Grouped, pivoted and normalized data and used K-Means Clustering algorithm to determine clusters based on the following variables (per 1,000 residents):**

Transposing the data:

```python
#taking relevant columns to be transposed
df_not_transposed = df_crime_np[['Crime Group2','Neighborhood','Latitude']]
#pivoting data
df_not_transposed = df_not_transposed.pivot_table(index='Neighborhood', columns='Crime Group2', values= 'Latitude', aggfunc='count')
#resetting index
df_not_transposed['Neighborhood'] = df_not_transposed.index
df_not_transposed.index = df_not_transposed.index.rename('index')
```

-image

get population sum for each neighborhood
```python
df_population = pd.DataFrame(df_nyc_pop.groupby('Neighborhood')['Population'].sum())
df_population['Neighborhood'] = df_population.index
df_population.index = df_population.index.rename('index')
```

merge dataframes and add qoutient in order to get crimes per 1,000 population
```python
df_crime_per_1000 = pd.merge(df_not_transposed,df_population,how='left',left_on=['Neighborhood'],right_on=['Neighborhood'])
df_crime_per_1000['qoutient'] = df_crime_per_1000['Population'] / 1000
```

<img src="/assets/img/with-qoutient.png">

Get crimes per 1,000
```python
cols = ['Non Violent','Theft','Violent','Drug','Traffic'] 
df_crime_per_1000 = df_crime_per_1000.fillna(0)
for col in cols:
    df_crime_per_1000[col] = df_crime_per_1000[col] / df_crime_per_1000.qoutient
```

```python
df_crime_per_1000 = df_crime_per_1000.replace([np.inf, -np.inf], np.nan)
df_crime_per_1000 = df_crime_per_1000.dropna()

df_crime_per_1000 = pd.merge(df_crime_per_1000, df_population[['Neighborhood','Borough']].drop_duplicates(),how = 'left',left_on=['Neighborhood'],right_on=['Neighborhood'])
```

<img src="/assets/img/transposed-df-final.png">


**Step 6: Perform K-Means Clustering**

Clustering is a form of unsupervised exploratory analysis that aims to segment data based on similar traits. In our case, we a used k-means clustering algorithm, where the number of clusters (K) is specified prior to clustering. In K-Means clustering, k (the selected number of clusters) centroids are chosen as random data points. Our data points are then taken and assigned to the nearest centroid based on Euclidean distance. Once all data points are assigned to a centroid, the mean of each cluster is calculated and determined to be the new centroids. All data points are then reassigned to the nearest new centroid value. This process continues until the data points stop changing clusters, and once this occurs the total variation amongst points in the clusters is then calculated. This entire process is repeated x amount of times and the sequence with the lowest variation is used to cluster the data.

- import libraries
```python
import seaborn as sns 
import plotly as py
import plotly.graph_objs as go
from sklearn.cluster import KMeans
from sklearn import preprocessing
import warnings
import os
from sklearn import preprocessing
warnings.filterwarnings("ignore")
py.offline.init_notebook_mode(connected = True)
#Declare dataset with our features for clustering
features = pd.DataFrame(df_crime_per_1000[['Non Violent','Theft','Violent','Drug','Traffic','Population']])
```

As K-Means uses Euclidean distance between data points, it's important that we standardize data so that our variables are on the same scale.

```python
#Normalize data
from sklearn import preprocessing
features_standardized = preprocessing.scale(features)
features_standardized = pd.DataFrame(features_standardized)
```

Next, we determine the optimal number of clusters to input into our algorithm. To do this, We'll graph out number of clusters on the x-axis and within-cluster-sum-of-squares (variation) on the y-axis.

```python
plt.figure(figsize=(10, 8))
wcss = []
for i in range(1, 15):
    kmeans = KMeans(n_clusters = i, init = 'k-means++')
    kmeans.fit(features_standardized)
    wcss.append(kmeans.inertia_) #criterion based on which K-means clustering works
plt.plot(range(1, 15), wcss)
plt.title('The Elbow Method')
plt.xlabel('Number of clusters')
plt.ylabel('WCSS')
plt.show()
```

<img src="/assets/img/elbow-method.png">

Looking at the graph, I selected 6 clusters

```python
#Fitting K-Means to the dataset
kmeans = KMeans(n_clusters = 6, init = 'k-means++')
y_kmeans = kmeans.fit_predict(features_standardized)
#beginning of  the cluster numbering with 1 instead of 0
y_kmeans1=y_kmeans+1
#New list called cluster
cluster = list(y_kmeans1)
#Adding cluster to our data set
df_crime_per_1000['cluster'] = cluster
#Mean of clusters 1 to 4
kmeans_mean_cluster = pd.DataFrame(round(df_crime_per_1000.groupby('cluster').mean(),1))
kmeans_mean_cluster
```

<img src="/assets/img/cluster-counts.png">

Looking at the cluster averages and cluster counts, it's clear we have outliers. Because the K-Means algorithm uses the mean of clusters, it's sensitive to outliers and does not perform well with them. Additionally, I'd like to have more meaningful clusters, and our outlier data seems to be of more or less the same variety - very high crime neighborhoods. I decided to group these outliers into 1 'High Crime' cluster, then create a dataset that excludes these neighborhoods and re-run the K-Means algorithm. 

Note: 'Non Violent' represents 'General Non-Violent' crime

<img src="/assets/img/new-clusters.png">

The cluster distribution looks a little better here, and we can identify characteristics of the clusters that go beyond just different degrees of similar features. The clusters are described as follows:

- Low Crime: Medium Population (73 neighborhoods)
- Low Crime: High Population (46 neighborhoods)
- Low-Med Crime: Low-Med Population (49 neighborhoods)
- Medium Crime: Medium Population (17 neighborhoods)
- High Violent/Theft/General: Mid Drugs/Traffic (10 neighborhoods)
- Extremely High Theft (3 neighborhoods)
- High Crime: Low Population (14 neighborhoods)

Here we have some clusters around degrees of crime and population, and some insight into neighborhoods with high violence and theft with mid level drug and traffic crime, or areas with very high theft. However, it does appear that generally when 1 type of crime is high in a neighborhood all types of crime are high as well. While that in itself is a bit of a relevation, particuralry with types of crime such as traffic, if the data set variables were less linear K-Means clustering could output more interesting clusters.

A cool way to visualize the data in python:

```python
from mnist.loader import MNIST
import tools
from pandas.plotting import parallel_coordinates
import plotly.express as px

fig = px.parallel_coordinates(df_clusters[['Drug', 'General Crime', 'Theft', 'Traffic', 'Violent','Population','clusters 2']], color="clusters 2", labels={"clusters 2": "cluster",
                "Drug": "Drug", "General Crime": "General Crime","Theft": "Theft", "Traffic": "Traffic",
                 "Violent": "Violent", "Population":"Population",},
                             color_continuous_scale=px.colors.diverging.Tealrose,
                             color_continuous_midpoint=2)
fig.show()
```

<img src="/assets/img/newplot.png">

**Step 7. Visualized data in Tableau**


[NYC Neighborhood Dashboard Link](https://public.tableau.com/views/NYCNeighborhoodCrimeTypeClustersv2/Dashboard15?:language=en&:display_count=y&:origin=viz_share_link)

Here's another dashboard I created with just the crime data.

<img src="/assets/img/NYC-Crime-Data.png">

[NYC Crime Dashboard Link](https://public.tableau.com/profile/kb8769#!/vizhome/NYCCrimeCoordinates/Dashboard1)

**Conculsions**
--->
