---
layout: post
title:  "NBA Team Win% Predictor"
date:   2020-07-28 00:16:41 -0400
categories: jekyll update
---
1. Developed model that predicts team win% based on 9 key variables
2. Built NBA Team dashboards with 15 years of trends around the 9 key statistics that impact win % filterable by team and year

(Steps further below)

Team Win % Predictor containing 9 key statistics with adjustable sliders:

<!---<img src="/assets/img/NBATeamPredict.png">

<iframe 
frameborder="0" 
height="730" 
width="920" 
scrolling="no" src="https://public.tableau.com/views/BBallWinPredictor/Dashboard32?:language=en&:display_count=y&:origin=viz_share_link:showVizHome=no&:embed=yes">
</iframe>

Team Data Dashboard with 15 years of trends around key statistics that impact win % filterable by team and year. 
- The big green or red numbers next to each metric indicate the team value 
- The smaller number to it's bottom right is the difference from the league average
  - If the number is red, the team is below league average
  - If the number is green, the team is above league average
<img src="/assets/img/NBATeam.png">

Dashboard can be accessed here (fullscreen recommended): https://public.tableau.com/views/BBallAnalysis/Dashboard1?:language=en&:display_count=y&:origin=viz_share_link


Part 1: Developed a model to predict team win % based on 9 key variables.

- NBA team data was pulled using identical webscraper in NBA Player Predictions
- Build out multivariate regression model:

Import libraries, format data
{% highlight ruby %}
import matplotlib.pyplot as plt
import seaborn as sns
import pylab

from scipy import stats
import statsmodels.api as sm
from statsmodels.stats import diagnostic as diag
from statsmodels.stats.outliers_influence import variance_inflation_factor

from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error, r2_score, mean_absolute_error

%matplotlib inline

df_rm = df_stats[['Year','Wins','Losses','Winning Percentage', 'Points Per Game','Opponent Points Per Game',
                  'Games Played','Average Field Goals Made', 'Average Field Goals Attempted','Field Goal Percentage',
                  'Average 3-Point Field Goals Made','Average 3-Point Field Goals Attempted','3-Point Field Goal Percentage',
                  'Average Free Throws Made','Average Free Throws Attempted','Free Throw Percentage','Offensive Rebounds Per Game',
                  'Defensive Rebounds Per Game','Rebounds Per Game','Assists Per Game','Steals Per Game','Blocks Per Game',
                  'Turnovers Per Game']]
data_floats=data_floats.astype(float)

data_floats.index = data_floats['Winning Percentage']
data_floats = data_floats.dropna(how='any',axis=0)
data_floats=data_floats.astype(float)
{% endhighlight %}

Look at correlation matrix and heatmap to determine appropriate variables to include:
{% highlight ruby %}
#print out a correlation matrix of our dataframe
corr = data_floats.corr()
display(corr)
#plot heatmap
sns.heatmap(corr,xticklabels = corr.columns, yticklabels = corr.columns, cmap='RdBu')
{% endhighlight %}

Check variance inflation factor and remove redundent variables that could potentially lead to incorrect coefficient values (typically, if > 5 it should be removed)
{% highlight ruby %}
data_before = df_stats
x1 = sm.tools.add_constant(data_before)
#create a series for both 
series_before = pd.Series([variance_inflation_factor(x1.values,i) for i in range(x1.shape[1])], index = x1.columns)
print("data before")
print('-'*100)
display(round(series_before),2)
#generally if above 5 drop it:
#Removing columns with mulitcollinearity
data_after = data_floats.drop(['Wins','Losses','Points Per Game','Games Played','Field Goal Percentage','Year',
                               'Offensive Rebounds Per Game','Defensive Rebounds Per Game','Average Free Throws Made',
                               'Opponent Points Per Game','Average Field Goals Made','Average Field Goals Attempted', 
                               'Average 3-Point Field Goals Attempted'],axis=1)
x2 = sm.tools.add_constant(data_after)
series_after = pd.Series([variance_inflation_factor(x2.values,i) for i in range(x2.shape[1])], index = x2.columns)
print("data After")
print('-'*100)
display(round(series_after, 2))
{% endhighlight %}

Taking the metrics from above into consideration, I decided to use the 9 variables below to build out the model. I wanted to only include the minimum number of key statistics as that a general manager can realistically impact with personnel decisions. (e.g Including additional small variables, such as opponents blocks per game, may help improve our model's accuracy, but from a GM's perspective will not provide actionable value)

9 Key variables:
 1/2. FG%/Opponent FG% 
 3/4. 3P%/Opponent 3P%
 5/6. 3P Made/Opponent 3P Made
 7/8. Assists/Opponent Assists
 9. Rebound Differential
 
{% highlight ruby %}
#Build the model: Define input variable and our output variable (x,y)
X = data_after.drop('Winning Percentage', axis=1)
Y = data_after[['Winning Percentage']]
#split dataset into training and testing portion
x_train, x_test, y_train, y_test = train_test_split(X,Y, test_size = .20, random_state = 1)
#create an instance of our model
regression_model = LinearRegression()
#fit the model
regression_model.fit(x_train,y_train)
y_predict = regression_model.predict(x_test)
{% endhighlight %}
