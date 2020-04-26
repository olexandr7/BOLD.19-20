

```python
#Importing libraries
import pandas
import csv
```


```python
#Reading csv files
population_df = pandas.read_csv('county_population.csv', encoding = 'utf8')
#print(population_df)
testresults_df = pandas.read_csv('opendata_covid19_test_results.csv', encoding = 'utf8')
#print(testresults_df)
```


```python
#Data quality check - weather NULLS are in required coulmns
pandas.isna(testresults_df).sum()
#627 tests don't have County specified
```




    id                      0
    Gender                  0
    AgeGroup              164
    Country                 0
    County                627
    ResultValue             0
    StatisticsDate          0
    ResultTime              0
    AnalysisInsertTime      0
    dtype: int64




```python
#Rows where County=NULL
testresults_df.loc[testresults_df['County'].isna()].Country.value_counts()
#4 tests from Estonia have no County specified
```




    Tundmatu    408
    Välismaa    215
    Eesti         4
    Name: Country, dtype: int64




```python
#Aggegating results by Counties
testresults_df_grouped = testresults_df.groupby('County')['ResultValue'].count().to_frame()
```


```python
#Joining 2 datasets (testing results and population numbers)
casesbycounty_df = pandas.merge(testresults_df_grouped, population_df, how='inner', left_on = 'County', right_on = 'CountyName')
```


```python
#Reordering and renaming columns
column_names = ['CountyName', 'ShortName', 'Population', 'ResultValue']
casesbycounty_df = casesbycounty_df.reindex(columns=column_names)
casesbycounty_df = casesbycounty_df.rename(columns={'ResultValue': 'TestsCount', 'Population': 'PopulationCount'})
```


```python
#Calculating percentage of tested population
casesbycounty_df['TestedPopulation'] = ((casesbycounty_df.TestsCount / casesbycounty_df.PopulationCount)*100)

#Adding column with Country name
casesbycounty_df['Country']='Estonia'
```


```python
#Writing the blended dataset to csv
casesbycounty_df.to_csv('tests_by_county.csv', sep='\t', encoding='utf-8', float_format='%11.6f')
```


```python
#Aggegating results by Counties + Gender
#casesbycountygender_df = testresults_df.groupby(['County','Gender'])['ResultValue'].count().to_frame()
#Writing the dataset to csv
#casesbycountygender_df.to_csv('tests_by_county_gender.csv', sep='\t', encoding='utf-8')
```
