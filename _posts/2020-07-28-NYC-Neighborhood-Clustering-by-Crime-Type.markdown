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
