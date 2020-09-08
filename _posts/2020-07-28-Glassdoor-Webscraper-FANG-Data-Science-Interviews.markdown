---
layout: post
title:  "Glassdoor Webscraper: FANG Data Science Interview Questions"
date:   2020-07-28 00:16:43 -0400
categories: jekyll update
---

# Project Overview
Compile Glassdoor Interview Questions and Data for FANG Data Science positions

# Libraries Used:
- pandas
- BeautifulSoup
- numpy
- urlopen
- requests
- collections

# Steps taken:
1. Webscrape Glassdoor Data
2. Clean and format data
3. Consolidate into Tableau Dashboard

**Step 1. Webscrape Glassdoor Data**

First got to the glassdoor and inspect the html code to find out where the data we care about is located.

example link: https://www.glassdoor.com/Interview/Facebook-Data-Scientist-Interview-Questions-EI_IE40772.0,8_KO9,23.htm

The interview question, experience, and offer data are located in classes rather than tables.

```python
import requests
from urllib.request import urlopen
from bs4 import BeautifulSoup
import numpy
import pandas as pd

#Create lists to scrape data into
fb_interview_questions_list = []
fb_interview_experience_list = []
fb_offer_exp_diff_list = []

url = https://www.glassdoor.com/Interview/Facebook-Data-Scientist-Interview-Questions-EI_IE40772.0,8_KO9,23.htm
headers= {'User-Agent': 'Mozilla/5.0'}
response = requests.get(url, headers = headers)
#Make sure web page exists (status code = 200)
if response.status_code != 200:
    print('No Glassdoor data for this job title/company combination available')
soup = BeautifulSoup(response.content, 'html.parser')

#Get interview Questions
for span in soup.find_all('span', {'class':'interviewQuestion'}):
    fb_interview_questions_list.append(span.text)

#Get interview summaries
for i in soup.find_all(class_ = 'interviewDetails continueReading interviewContent mb-xsm'):
    fb_interview_experience_list.append(i.text)

#Offer/No Offer, Positive/Negative Exp, Easy/Avg/Difficult Interview
for span in soup.find_all('span', {'class':'middle'}):
    fb_offer_exp_diff_list.append(span.text)
```

This is for just page 1 on Glassdoor. There are usually more, and for big companies typically hundreds. 

Page 1:
- https://www.glassdoor.com/Interview/Facebook-Data-Scientist-Interview-Questions-EI_IE40772.0,8_KO9,23.htm

Page 2:
- https://www.glassdoor.com/Interview/Facebook-Data-Scientist-Interview-Questions-EI_IE40772.0,8_KO9,23**_IP2**.htm
