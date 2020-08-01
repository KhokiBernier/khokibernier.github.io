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

- pandas
- numpy
- BeautifulSoup
- urllib
- sklearn
- matplotlib

The following steps were taken, and will be elaborated on with code snippets further below: 

1. Webscrapped and wrangled 15+ years of NBA team and player data from basketball-reference.com and espn.com.
2. Built multivariate regression and random forest regresor models for key basketball statistics (points, rebounds, assists, etc) using data pulled in previous setp.
3. Built a prediction matrix for each basketball statistic and age that calculates league avg percent change.
4. Conditionally applied prior year weighted averages based on age.
5. For rookies - data was pulled from sports-reference.com and only multivariate regression on college stats was used.
6. Determined which of the 4 methods (regression, random forest, avg % change, and prior year weighted avgs) is most accurate by looking at RMSE and use it for the specific statistic in question for 2020 player predictions. While regression was usually the most accurate individual method, in most cases averaging the predicition values of all 4 methods returned the lowest RMSE.
7. Pulled ESPN's 2020 season projections from web and merged into a dataframe with my predictions and actual values.
8. Loaded data into Tableau and calculated % Accuracy as well as RMSE, and performed visual analysis

My Model vs ESPN's Projections below: 

<img src="/assets/img/KBVESPN.jpg">

! [image tooltip here](/assets/img/KBVESPN.jpg)

**Step 1: Webscrapped and wrangled 15+ years of NBA team and player data from basketball-reference.com and espn.com.**

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

**Step 2: Built multivariate regression and random forest regresor models for key basketball statistics (points, rebounds, assists, etc) using data pulled in previous setp.**

Regression Model:

Import libraries and create RMSE Function
{% highlight ruby %}
import matplotlib.pyplot as plt
import math
import pylab

from scipy import stats
import statsmodels.api as sm

from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error, r2_score, mean_absolute_error

%matplotlib inline

#function to calculate rmse (root mean squared error)
def rmse(actual, prediction):
    mse = mean_squared_error(actual,prediction)
    rmse = math.sqrt(mse)
    return(rmse)
{% endhighlight %}

Build Model and evaluate RMSE:
{% highlight ruby %}
data_ppg = df_stats[['Age','PTS','PTS_1yp','PTS_2yp','PTS_3yp','MP_1yp','MP_2yp','MP_3yp','PG','SG','SF','PF','Center','Traded','Starter','Years_Pro_Now']]
#Build the model: Define input variable and our output variable (x,y)
X_ppg = data_ppg.drop('PTS', axis=1)
Y_ppg = data_ppg[['PTS']]
#split dataset into training and testing portion
x_train_ppg, x_test_ppg, y_train_ppg, y_test_ppg = train_test_split(X_ppg,Y_ppg, test_size = .20, random_state = 1)
#create an instance of our model
rm_ppg = LinearRegression()
#fit the model
rm_ppg.fit(x_train_ppg,y_train_ppg)
y_predict_ppg = rm_ppg.predict(x_test_ppg)
#evaluate model
X2 = sm.add_constant(X_ppg)
#create OLS model
model_ppg = sm.OLS(Y_ppg,X2)
est_ppg = model_ppg.fit()
print(est_ppg.summary())
#check for normality of residuals
sm.qqplot(est_ppg.resid,line='s')
pylab.show()
print("RMSE: {:.4}".format(rmse(y_test_ppg, y_predict_ppg)))
{% endhighlight %}

<img src="/assets/img/R2.png">
<img src="/assets/img/Residual-Graph.png">
- note: the deviation of the residuals at the top right of the graph indicates heteroscedasticity

Random Forest Regressor Model:
<img src="/assets/img/RF.png">

**Step 3: Built a prediction matrix for each basketball statistic and age that calculates league avg percent change** (e.g. if the average player's points per game increases by 8% from when they start the season at age 24 versus age 25, the points prediction for a player turning 25 is 8% more than prior year)

- Building Prediction Matrix:
{% highlight ruby %}
###Build Matrix:
#Variable to store all player percent change values for each stat
pcnt_change_agg = numpy.arange(15).reshape(1,15)
#loop through each player's career stats and get percent change by year
for player in df_stats['Player'].unique():
    #storing ages
    cnt = df_stats[df_stats.Player ==player].Player.count()
    ages = numpy.asarray(df_stats[df_stats.Player ==player]['Age'].tolist())
    ages = numpy.asarray(ages).reshape(cnt,1)
    #storing percent change for key stats (14 total)
    pcnt_change = df_stats.loc[df_stats.Player ==player][['MP','FGA','FG%','3PA','3P%','eFG%','FTA','FT%','TRB','AST','STL','BLK','TOV','PTS']].replace('','0',regex=True).astype(float).pct_change()
    pcnt_change_array = numpy.asarray(pcnt_change)
    #concatenate ages and pcnt change, then add to aggregate list
    pcnt_change_array = numpy.concatenate([ages,pcnt_change_array], axis=1)
    pcnt_change_agg = numpy.concatenate((pcnt_change_array,pcnt_change_agg), axis=0)
#add column headers to matrix
columns = ['Age','MP','FGA','FG%','3PA','3P%','eFG%','FTA','FT%','TRB','AST','STL','BLK','TOV','PTS']
columns_array = numpy.asarray(columns).reshape(1,15)
pcnt_change_agg = numpy.concatenate([columns_array,pcnt_change_agg], axis=0)
{% endhighlight %}

{% highlight ruby %}
###convert to dataframe
df_pc_matrix = pd.DataFrame(pcnt_change_agg)
df_pc_matrix.columns = df_pc_matrix.iloc[0]
df_pc_matrix = df_pc_matrix[1:]
df_pc_matrix = df_pc_matrix.astype(float)
#Remove values where pcnt change could not be calculated (e.g a player's rookie year)
df_pc_matrix= df_pc_matrix.replace([numpy.inf, -numpy.inf], numpy.nan)
df_pc_matrix = df_pc_matrix.dropna()
{% endhighlight %}
{% highlight ruby %}
###Create Matrix
df_matrix = pd.DataFrame()
#loop through stat columns and get avgerage pcnt change for each age
for i in columns[1:15]:
    df_matrix[i] = df_pc_matrix.groupby('Age')[i].mean()
{% endhighlight %}

