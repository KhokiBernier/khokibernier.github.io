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


# NYC Neighborhoods Latin Population Breakdown - Latin Inspired Theme
<img src="/assets/img/NYC-Latin-Black.png">
<img src="/assets/img/NYC-Latin-Yellow.png">

# NYC Neighborhoods Cool Visual
<iframe frameborder="0" height="700" width="1050" scrolling="no" src="https://public.tableau.com/views/NYCNeighborhoods/Dashboard2?:language=en&:display_count=y&publish=yes&:origin=viz_share_link:showVizHome=no&:embed=yes"> </iframe>
