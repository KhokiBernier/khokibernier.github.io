---
layout: post
title:  "NBA Stat Predictor"
date:   2020-07-28 00:16:41 -0400
categories: jekyll update
---
Built a model to predict NBA Player Season Stats that outperformed ESPN's 2019-2020 season projections (to date, the season is still going on). The model was built in Python3 and projections were made using the following:

- Multivariate Regression
- Random Forest Regressor
- Average Percent Change by Age
- Prior Year Weighted Averages

The following libraries were primarily used:

- Pandas
- Numpy
- BeautifulSoup
- urllib
- sklearn
- matplotlib

The following steps were taken, and will be elaborated on with code snippets further below: 

1. Webscrapped and wrangled 15+ years of NBA team and player data from basketball-reference.com and espn.com.
2. Built multivariate regression and random forest regresor models for key basketball statistics (points, rebounds, assists, etc) using data pulled in previous setp.
3. Built a prediction matrix for each basketball statistic and age that calculates league avg percent change (e.g. if the average player's points per game increases by 8% from when they start the season at age 24 versus age 25, the points prediction for a player turning 25 is 8% more than prior year).
4. Conditionally applied prior year weighted averages based on age.
5. For rookies - data was pulled from sports-reference.com and only multivariate regression on college stats was used.
6. Determined which of the 4 methods (regression, random forest, avg % change, and prior year weighted avgs) is most accurate by looking at RMSE and use it for the specific statistic in question for 2020 player predictions. While regression was usually the most accurate individual method, in most cases averaging the predicition values of all 4 methods returned the lowest RMSE.
7. Pulled ESPN's 2020 season projections from web and merged into a dataframe with my predictions and actual values.
8. Loaded data into Tableau and calculated % Accuracy as well as RMSE, and performed visual analysis

My Model vs ESPN's Projections below: 

<img src="/assets/img/KBVESPN.jpg">

! [image tooltip here](/assets/img/KBVESPN.jpg)

Step 1: Webscrapped and wrangled 15+ years of NBA team and player data from basketball-reference.com and espn.com.

- Performed with BeautifulSoup and urlib libraries. 
- Looped through the url with the year as a variable
- Parsed through the html to find the proper table
- Appended rows and colums to lists, then converted to numpy array and subsequently a dataframe
- Cleansed the data and added the folllowing variables:
  - Position (binary dummy variables)
  - Traded? (Different team from last year)
  - Starter or Bench Player (40 Game threshold)
  - Years Pro
  - Stats from 1, 2, and 3 years prior for applicable players
  - For college players - Strength of Schedule

Webscraper code:
{% highlight ruby %}
import requests
from urllib.request import urlopen
from bs4 import BeautifulSoup
import numpy
import pandas as pd

year_to_loop = ['2004','2005','2006','2007','2008','2009','2010','2011','2012','2013','2014','2015','2016','2017','2018','2019']
years = []
players_stats_list_agg = []

#Loop through 15 years
for year in year_to_loop:
    url = 'https://www.basketball-reference.com/leagues/NBA_'+year+'_per_game.html'
    headers= {'User-Agent': 'Mozilla/5.0'}
    response = requests.get(url, headers = headers)
    soup = BeautifulSoup(response.content, 'html.parser')
    stat_table = soup.find_all('table', class_ = 'stats_table')
    stat_table = stat_table[0]
    
    #append players and stats into players_stats_list
    players_stats_list = []
    for row in stat_table.find_all('tr'):
        for cell in row.find_all('td'):
            players_stats_list_agg.append(cell.text)
            players_stats_list.append(cell.text)    
    
    #append column headers into headers_list
    headers_list = [th.getText() for th in soup.findAll('tr',limit=2)[0].findAll('th')]
    for i in range(int((int(len(players_stats_list)))/(int(len(headers_list)-1)))):
        years.append(year)
{% endhighlight %}

- I then converted lists -> arrays -> reshaped and concatenated arrays -> convert to dataframe

The following snippet shows how I added additional variables with an example of starter/benchplayer:
{% highlight ruby %}
df_stats['Starter'] = (df_stats.GS.astype(int) > 40).astype('int')
{% endhighlight %}

Step 2: Built multivariate regression and random forest regresor models for key basketball statistics (points, rebounds, assists, etc) using data pulled in previous setp.



