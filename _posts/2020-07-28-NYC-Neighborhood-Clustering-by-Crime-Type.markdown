---
layout: post
title:  "NYC Neighborhood Clustering by Crime Type"
date:   2020-07-28 00:16:42 -0400
categories: jekyll update
---

Documentation in progress..

**Project Overview**

Clustered New York City nieghborhoods by crime type per 1,000 residents and neighborhood population using K-Means algorithm in Python and visualized data in Tableau


<img src="/assets/img/NYC-Neighborhood-Dashboard.png">

(input link to define clustering)

**Libraries used:**

- pandas
- numpy
- matplotlib
- seaborn
- sklearn
- plotly

**Links**

- NYC Arrest dataset: https://data.cityofnewyork.us/Public-Safety/NYPD-Arrest-Data-Year-to-Date-/uip8-fykc
- Code to get Neighborhood and venues: https://codekarim.com/node/57 **(Note: I do not take credit for this code)**
- NYC Neighborhood by population file: https://data.cityofnewyork.us/City-Government/New-York-City-Population-By-Neighborhood-Tabulatio/swpk-hqdp
- Missing Neighborhood Populations: https://popfactfinder.planning.nyc.gov/#11.37/40.7583/-74.0043 
- NYC Map to verify Midtown South: https://maps.nyc.gov/crime/

**Steps Taken**

1. Downloaded and cleaned file that contained 5 million+ rows of NYC crime occurances, descriptions, and coordinates
2. Pulled NYC neighborhood coordinates as well as coordinates from all venues in each neighborhood by following steps in this link https://codekarim.com/node/57 
3. Mapped crime location to neighborhood by finding minimum Euclidean distance between crime location and all venues
4. Mapped neighborhood population to nieghborhood using 'NYC Neighborhood by population file' dataset
  - Built several functions to map neighborhoods, as dataset nieghborhood syntax was not the same
5. Grouped, pivoted and normalized data and used K-Means Clustering algorithm to determine clusters based on the following variables:
- Violent Crime / 1,000 residents
- Theft / 1,000 residents
- Drug Related Crime / 1,000 residents
- Traffic Crime / 1,000 residents
- General Non-Violent Crime / 1,000 residents (loittering, gambling, health code violations, etc)
6. Visualize data in Tableau

**Step 1: Downloaded and cleaned file that contained 5 million+ rows of NYC crime occurances, descriptions, and coordinates**

- Dataset taken from this link: https://data.cityofnewyork.us/Public-Safety/NYPD-Arrest-Data-Year-to-Date-/uip8-fykc
- Cleaned data

{% highlight ruby %}
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
{% endhighlight %}

Look at the distinct offense description values and determine groupings:

{% highlight ruby %}
#listing them out this way makes them easy to copy paste
df_crime.OFNS_DESC.unique()
{% endhighlight %}

{% highlight ruby %}
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
        crime_group2.append('Violent')
    elif (i in non_violent) == True:
        crime_group2.append('Non Violent')
    elif (i in traffic) == True:
        crime_group2.append('Traffic')
    elif (i in theft) == True:
        crime_group2.append('Theft')
    elif (i in drugs) == True:
        crime_group2.append('Drug')
    else:
        crime_group2.append('Other')
        
df_crime['Crime Group'] = crime_group_list
{% endhighlight %}
