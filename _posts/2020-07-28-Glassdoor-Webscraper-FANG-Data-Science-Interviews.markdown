---
layout: post
title:  "Glassdoor Webscraper: FANG Data Science Interview Questions"
date:   2020-07-28 00:16:43 -0400
categories: jekyll update
---

# Project Overview
Visualize and detail NYC Latin/Hispanic Neighborhood population by nationality.

# NYC Neighborhoods Latin Population Breakdown
White

<iframe frameborder="0" height="800" width="1400" scrolling="no" src="https://public.tableau.com/views/NYCLatinHispanicPopulationBreakdown/Dashboard1?:language=en&:display_count=y&:origin=viz_share_link:showVizHome=no&:embed=yes"> </iframe>

Dark
<iframe frameborder="0" height="800" width="1400" scrolling="no" src="https://public.tableau.com/views/NYCLatinHispanicPopulationBreakdown-Dark/Dashboard13?:language=en&:display_count=y&:origin=viz_share_link:showVizHome=no&:embed=yes"> </iframe>


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
3. Reformat Data in Order to Visualize in Tableau

**Step 1. Pivot Latin/Hispanic breakdown dataset so that it can better analyzed in Tableau**

'''python

'''
