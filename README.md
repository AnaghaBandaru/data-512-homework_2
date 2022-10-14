# Considering Bias in Data

## Goal

The goal of this assignment is to explore the concept of bias in data using Wikipedia articles. We consider articles on political figures from different countries which is combined with a dataset of the country populations, and use a dataset created from calling the API for a machine learning service called ORES to estimate the quality of each article. We then perform an analysis of how the coverage of politicians on Wikipedia and the quality of articles about politicians varies among countries.

## Data Source

The politicians dataset was generated by crawling the [The Wikipedia Category:Politicians by nationality](https://en.wikipedia.org/wiki/Category:Politicians_by_nationality).
The population dataset is drawn from [the world population data sheet](https://www.prb.org/international/indicator/population/table/) published by the Population Reference Bureau.
The revision id for each article is required to generate the quality scores and these have been brought in from [Mediawiki Action API](https://www.mediawiki.org/wiki/API:Info)

The scores for each article are generated by using [ORES](https://www.mediawiki.org/wiki/ORES), which is a web service and API that provides machine learning as a service.
Link to API Documentation : https://www.mediawiki.org/wiki/ORES

## Project Structure

```bash
data-512-homework_2
├── data
│   ├── articles_no_revId.txt
│   ├── wp_countries-no_match.txt
│   ├── wp_politicians_by_country.csv
│   └── population_by_country_2022.csv
│   └── politicians_by_country_2022.csv
├── source
│   ├── DataAcquisitionAndAnalysis.ipnyb
├── LICENSE
└── README.md
 ```

## File Descriptions

**data** : 

- *articles_no_revId.txt* : This file is generated in the DataAcquisitionAndAnalysis.ipnyb notebook and contains the list of articles that did not return a revisionId, these articles have been removed from further analysis.
- *wp_countries-no_match.txt* : This file is generated in the DataAcquisitionAndAnalysis.ipnyb notebook and contains the list of countries from the politicians dataset that do not have the corresponding values in the population dataset (Korean) or countries in the populations dataset that did not have any articles in the politicians dataset.
- *wp_politicians_by_country.csv* : This file is generated in the DataAcquisitionAndAnalysis.ipnyb notebook and contains politician, population, revision id and article quality information. 
Schema: 
    
    | Columns           | Description                                     |
    | ------------------| ----------------------------------------------- |
    | country           | country of the politician                       |
    | region            | region the country belongs to                   |
    | population        | population of the country                       |
    | article_title     | name of the politician                          |
    | revision_id       | revision id of teh wikipedia article            |
    | article_quality   | Article Quality Prediction from ORES            |

- *politicians_by_country_2022.csv* : This file is generated in the DataAcquisitionAndAnalysis.ipnyb notebook and contains monthly data for mobile-app and mobile-web access type.

    | Columns | Description                                     |
    | ------- | ----------------------------------------------- |
    | name    | name of the politician                          |
    | url     | link to the wikipedia article                   |
    | country | country of the politician                       |

- *population_by_country_2022.csv* : This file is generated in the DataAcquisitionAndAnalysis.ipnyb notebook and contains monthly data of all the access types with a cumulative sum of the page traffic.
    | Columns               | Description                                                            |
    | --------------------- | ---------------------------------------------------------------------- |
    | Geography             | Region and Country Hierarchy                                           |
    | Population (millions) | Population of countries & cumulative population of regions             |

**source**:

- *DataAcquisitionAndAnalysis.ipnyb* : In this notebook, I bring in the data from different sources before following the steps detailed in process for Analysis. 

## Process followed for the assignment
This is the process I followed for the assignment. 

1. Data Acquisition:  I loaded the politician and population csv files before using the Wikimedia API to get the revision Id of each article. Usinf this revision id, I then called the ORES API to the the quality prediction for the corresponding articles.

2. Data Inconsistencies: 
    - Politician Dataset : 2 records are entirely duplicated and 46 articles/politicians have more than 1 records (with different country values but the same link is provided). I removed the completely duplicated records but did not process the 46 articles with multiple records as I had to deal with it in a case-by-case basis. Further details are provided in the jupyter notebook.
    - Population Dataset : 6 countries had 0 population and I removed these countries from the final dataset.
    - Revision id : On requesting the revision id, 7 articles did not return one and I removed these records from the final dataset.
    - Combining the Politician and Population datasets : 1 country present in the politician dataset was not present in the population dataset (Korean), I removed the articles corresponding to this country. 25 countries present in the population dataset did not have any corresponding articles in the politician dataset and I also removed these records. A list of all these countries is provided in the wp_countries-no_match.txt file.

3. Basic Data Processing: 
    - In the politician dataset, I renamed the columns and dropped the url column after retrieving the revision ids.
    - In the population dataset, the countries and regions are provided in a hierarchical manner in the 'Geography' column with regions in capital case, I extracted the region values and created a new column in the dataset with the matching region values for a country. I retained the region at the lowest hierarchical level provided. There are a total of 18 regions

5. Preparing Master Dataset
    -  I combined the politician and population dataset with the corresponding revision id's and article quality scores to generate the final dataset. This dataset contained 7474 articles and 203 countries.

6. Analysis
    - For the analysis, I created two tables for the country and region ranking.
    - For the country ranking, I first calculated the count of total and high quality articles for each country and then calculated the per capita values using the corresponding population values for each country. I did not convert the values to millions as the ranking won't be affected.
    - For the region ranking, I aggregated the count of total and high quality articles for all the countries in a region and loaded the corresponding population values provided by region in the initial population dataset, I then calculated the per capita values.

7. Results
    - I generated the required results using the two tables created above by sorting the data either by total article or high quality count and then using the head() function to get the required number of values. 



## Research Implications
1. What biases did you expect to find in the data (before you started working with it),
and why?

    I expected the western world countries to have a higher number of well written articles (article ORES score is 'GA' or 'FA'). Because we are getting the data from English Wikipedia, we can expect it to be biased to english speaking countries. English speakers would be less acquainted with politicians from non-english speaking countries and hence, we can expect less information/edits for the politicians from those countries. I also expected to see bias due to population - countries with a larger population could have higher quality articles for their politicians but since we are considering per capita values for the ranking, this bias should be mitigated.

2. What (potential) sources of bias did you discover in the course of your data
processing and analysis?

    We see that the bottom 10 country list by both total coverage and high quality article coverage consists of all non-english speaking countries, this could indicate a high language bias.
    We also find that the top 10 country list generally consists of countries with really low populations (mean for population is around 38 million and all the values foor population in the top 10 list are below 6 million), this indicates that there is a per capita bias.This is probably not the best way to account for varying population sizes for the countries because we do not see an obvious trend between the number of articles and the population of a country. To account for this, we can calculate the average per capita article coverage for all countries and discount that from the values.

3. Can you think of a realistic data science research situation where using these data
(to train a model, perform a hypothesis-driven research, or make business
decisions) might create biased or misleading results, due to the inherent gaps and
limitations of the data?

    Let us assume, a research group was trying to find the most relevant and popular politicians in Asia by using this data. Assuming they include the predicted ORES score for the politicians for their ranking - with politicians with higher quality wikipedia articles having a higher rank, their results would be heavily biased as the training model would represent the english-speaking countries far more than other countries. Using this data would generate a very biased ranking as the data used in the training model is representing a different domain  to the domain of data they would be testing on.
