---
layout: post
title:  "NYC Latin/Hispanic Population Breakdown"
date:   2020-07-28 00:16:44 -0400
categories: jekyll update
---
Documentation in progress

# Project Overview
Visualize and detail NYC Latin/Hispanic Neighborhood population by nationality.

# NYC Neighborhoods Latin Population Breakdown
<img src="/assets/img/NYC-Latin-White.png">

# Motivation
- A lof of latin/hispanics immigrate to New York City and not all are able to speak English. Living in a neighborhood with people from your country can help with the transition and assimilation into a new country while also making you feel at home. 
- I've always considered moving to New York City and would like to live in a Spanish speaking area
- Cultures between Spanish speaking countries can differ vastly, so breaking down latin/hispanic into nationality is more helpful

# Libraries Used:
- pandas
- statistics

# Links to Datasets:
- Latin/Hispanic Breakdown: https://www1.nyc.gov/site/planning/planning-level/nyc-population/american-community-survey.page
- NYC Neighborhood Coordinates: https://data.cityofnewyork.us/City-Government/Neighborhood-Tabulation-Areas-NTA-/cpf4-rkhq

# Steps taken:
1. Pivot Latin/Hispanic breakdown dataset so that it can better analyzed in Tableau
2. Clean MULTIPOLYGON column and obtain avg latitiude and longitude for each neighborhood from coordinates dataset
3. Merge datasets
4. Visualize in Tableau

**Step 1. Pivot Latin/Hispanic breakdown dataset so that it can better analyzed in Tableau**

The original dataset has 484 columns with confusing headers, so the first step was to identify the columns we need and then rename them.

{% highlight ruby %}
df.rename(columns = {    
    'GeogName': 'Neighborhood',
    'Pop_1E': 'Total Population',
    'Hsp1E': 'Total Hispanic/Latino',
    'HspMeE': 'Mexican',
    'HspPRE': 'Puerto Rican',
    'HspCubE': 'Cuban',
    'HspDomE':'Dominican',
    'HspCAmE': 'Central American',
    'HspCRE': 'Costa Rican',
    'HspGuatmE':'Guatemalan',
    'HspHndM': 'Honduran',
    'HspNicE':'Nicaraguan',
    'HspPanE':'Panamanian',
    'HspSalvE':'Salvadoran',
    'HspSAmE':'South American',
    'HspArgE': 'Argentinean',
    'HspBolE': 'Bolivian',
    'HspChlE': 'Chilean',
    'HspColE': 'Colombian',
    'HspEcE': 'Ecuadorian',
    'HspPrgE': 'Paraguayan',
    'HspPeruE': 'Peruvian',
    'HspUrgE': 'Uruguayan',
    'HspVnzE': 'Venezuelan',
    'HspOthE': 'Total Other Hispanic/Latino',
    'HspSpnrdE': 'Spaniard',
    'HspSpnshE': 'Spanish',
    'HspSpAmE': 'Spanish American',
    'HspAllOthE': 'Other Hispanic/Latino'        
}, inplace=True)

df_lap = df[['Neighborhood','Borough','GeoID','Total Population','Total Hispanic/Latino','Mexican','Puerto Rican','Cuban','Dominican',
'Central American','Costa Rican','Guatemalan','Honduran','Nicaraguan','Panamanian','Salvadoran','South American',
'Argentinean','Bolivian','Chilean','Colombian','Ecuadorian','Paraguayan','Peruvian','Uruguayan','Venezuelan',
'Total Other Hispanic/Latino','Spaniard','Spanish','Spanish American','Other Hispanic/Latino']]
{% endhighlight %}

Our new dataframe has 31 columns with much more intuitive headers. This'll make performing analysis much easier. 

The next step is to transpose/pivot the data. Right now we have each Nationality (28 total) as it's own column, and our 195 neighborhoods as all our rows. After we transpose/pivot the data we'll remove each specific nationality column (Dominican, Guatemalan, etc), and replace it with 1 'Nationality' column. We'll keep our 195 neighborhood rows, and have 195 neighborhood times 28 nationalitys per neighborhood = 5460 rows. The purpose of doing this is that it'll allow us to perform more advanced analysis and aggregations/filtering in Tableau. 

{% highlight ruby %}
#Dropping GeoID and Borough (we'll re-add later) and switching Nationalities to rows and Neighborhoods to columns
df_la_t = df_la.drop(['GeoID','Borough'],axis=1).T
df_la_t.index = df_la_t.index.rename('Nationality')
#Replacing numeric column indexes with Neighborhood indexes
df_la_t.columns = df_la_t.iloc[0]
df_la_t = df_la_t[1:]
df_la_t
{% endhighlight %}

Results in the following:
-transposed df photo

Next, I looped through the dataframe columns (Neighborhoods) and nested a loop through each nationality (index) and value (values)

{% highlight ruby %}
#Dropping GeoID and Borough (we'll re-add later) and switching Nationalities to rows and Neighborhoods to columns
df_la_t = df_la.drop(['GeoID','Borough'],axis=1).T
df_la_t.index = df_la_t.index.rename('Nationality')
#Replacing numeric column indexes with Neighborhood indexes
df_la_t.columns = df_la_t.iloc[0]
df_la_t = df_la_t[1:]
df_la_t
{% endhighlight %}

-photo df-pivoted

We see 5,460 rows, which matches what we were expecting.
We can now merge it to our original dataframe to bring back the 'Borough' and 'GeoID' columns

{% highlight ruby %}
df_la_pivoted = pd.merge(df_la_pivoted, df_la[['Neighborhood','GeoID','Borough']], how = 'inner',left_on=['Neighborhood'],right_on=['Neighborhood'])
{% endhighlight %}

**Step 2. Clean MULTIPOLYGON column and obtain avg latitiude and longitude for each neighborhood from coordinates dataset**

While our dataset is complete enough to analyze in Tableau, I wanted to be able to plot out the neighborhoods on a map and in order to do so we need the coordinates of each Neighborhood.

**Step 3. Merge datasets**


**Step 4. Visualize in Tableau**



# NYC Neighborhoods Latin Population Breakdown - Latin Inspired Theme
<img src="/assets/img/NYC-Latin-Black.png">
<img src="/assets/img/NYC-Latin-Yellow.png">

# NYC Neighborhoods Cool Visual
<iframe frameborder="0" height="700" width="1050" scrolling="no" src="https://public.tableau.com/views/NYCNeighborhoods/Dashboard2?:language=en&:display_count=y&publish=yes&:origin=viz_share_link:showVizHome=no&:embed=yes"> </iframe>
