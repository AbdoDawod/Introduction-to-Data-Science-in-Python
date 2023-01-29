# Assignment 3
All questions are weighted the same in this assignment. This assignment requires more individual learning then the last one did - you are encouraged to check out the [pandas documentation](http://pandas.pydata.org/pandas-docs/stable/) to find functions or methods you might not have used yet, or ask questions on [Stack Overflow](http://stackoverflow.com/) and tag them as pandas and python related. All questions are worth the same number of points except question 1 which is worth 17% of the assignment grade.

**Note**: Questions 2-13 rely on your question 1 answer.


```python
import pandas as pd
import numpy as np
import re
# Filter all warnings. If you would like to see the warnings, please comment the two lines below.
import warnings
warnings.filterwarnings('ignore')

df1 = pd.read_excel('assets/Energy Indicators.xls', sheet_name='Energy', header=None,skiprows=18)[[2,3,4,5]]#, usecols = ['3'])
df1.columns = ['Country', 'Energy Supply', 'Energy Supply per Capita', '% Renewable']
df1 = df1[:227]
df1['Energy Supply']= df1['Energy Supply']*1000000
df1.replace('...', np.nan, inplace=True)
df1['Country'] = df1['Country'].apply(lambda x: re.sub(r'\d+$', '', x))
df1['Country'] = df1['Country'].apply(lambda x: re.sub(r'\(.*\)', '', x))
df1['Country'] = df1['Country'].str.strip()

mydict={"Republic of Korea": "South Korea", "United States of America": "United States", "United Kingdom of Great Britain and Northern Ireland": "United Kingdom", "China, Hong Kong Special Administrative Region": "Hong Kong"}
Energy =  (df1.replace({'Country': mydict}))
GDP = pd.read_csv('assets/world_bank.csv', skiprows=4)
gdp_dict = {"Korea, Rep.": "South Korea",  "Iran, Islamic Rep.": "Iran", "Hong Kong SAR, China": "Hong Kong"}
GDP =  (GDP.replace({'Country Name': gdp_dict})
        .rename(columns={'Country Name':'Country'}))
ScimEn = pd.read_excel('assets/scimagojr-3.xlsx')
```

### Question 1
Load the energy data from the file `assets/Energy Indicators.xls`, which is a list of indicators of [energy supply and renewable electricity production](assets/Energy%20Indicators.xls) from the [United Nations](http://unstats.un.org/unsd/environment/excel_file_tables/2013/Energy%20Indicators.xls) for the year 2013, and should be put into a DataFrame with the variable name of **Energy**.

Keep in mind that this is an Excel file, and not a comma separated values file. Also, make sure to exclude the footer and header information from the datafile. The first two columns are unneccessary, so you should get rid of them, and you should change the column labels so that the columns are:

`['Country', 'Energy Supply', 'Energy Supply per Capita', '% Renewable]`

Convert `Energy Supply` to gigajoules (**Note: there are 1,000,000 gigajoules in a petajoule**). For all countries which have missing data (e.g. data with "...") make sure this is reflected as `np.NaN` values.

Rename the following list of countries (for use in later questions):

```"Republic of Korea": "South Korea",
"United States of America": "United States",
"United Kingdom of Great Britain and Northern Ireland": "United Kingdom",
"China, Hong Kong Special Administrative Region": "Hong Kong"```

There are also several countries with parenthesis in their name. Be sure to remove these, e.g. `'Bolivia (Plurinational State of)'` should be `'Bolivia'`.

Next, load the GDP data from the file `assets/world_bank.csv`, which is a csv containing countries' GDP from 1960 to 2015 from [World Bank](http://data.worldbank.org/indicator/NY.GDP.MKTP.CD). Call this DataFrame **GDP**. 

Make sure to skip the header, and rename the following list of countries:

```"Korea, Rep.": "South Korea", 
"Iran, Islamic Rep.": "Iran",
"Hong Kong SAR, China": "Hong Kong"```

Finally, load the [Sciamgo Journal and Country Rank data for Energy Engineering and Power Technology](http://www.scimagojr.com/countryrank.php?category=2102) from the file `assets/scimagojr-3.xlsx`, which ranks countries based on their journal contributions in the aforementioned area. Call this DataFrame **ScimEn**.

Join the three datasets: GDP, Energy, and ScimEn into a new dataset (using the intersection of country names). Use only the last 10 years (2006-2015) of GDP data and only the top 15 countries by Scimagojr 'Rank' (Rank 1 through 15). 

The index of this DataFrame should be the name of the country, and the columns should be ['Rank', 'Documents', 'Citable documents', 'Citations', 'Self-citations',
       'Citations per document', 'H index', 'Energy Supply',
       'Energy Supply per Capita', '% Renewable', '2006', '2007', '2008',
       '2009', '2010', '2011', '2012', '2013', '2014', '2015'].

*This function should return a DataFrame with 20 columns and 15 entries, and the rows of the DataFrame should be sorted by "Rank".*


```python
def answer_one():
    # YOUR CODE HERE
    global Energy, GDP, ScimEn
    combined_df = pd.merge(pd.merge(Energy, GDP, on='Country'), ScimEn, on='Country')
    combined_df = combined_df.set_index('Country')
    combined_df=combined_df[['Rank', 'Documents', 'Citable documents', 'Citations',
                               'Self-citations', 'Citations per document', 'H index',
                               'Energy Supply', 'Energy Supply per Capita', '% Renewable',
                               '2006', '2007', '2008', '2009', '2010', '2011', '2012', '2013',
                               '2014', '2015']]
    combined_df = combined_df[combined_df['Rank'] <= 15]
    combined_df.sort_values(['Rank'], ascending=[1],inplace=True)
    return combined_df
    raise NotImplementedError()
```


```python
assert type(answer_one()) == pd.DataFrame, "Q1: You should return a DataFrame!"

assert answer_one().shape == (15,20), "Q1: Your DataFrame should have 20 columns and 15 entries!"

```


```python
# Cell for autograder.

```

### Question 2
The previous question joined three datasets then reduced this to just the top 15 entries. When you joined the datasets, but before you reduced this to the top 15 items, how many entries did you lose?

*This function should return a single number.*


```python
%%HTML
<svg width="800" height="300">
  <circle cx="150" cy="180" r="80" fill-opacity="0.2" stroke="black" stroke-width="2" fill="blue" />
  <circle cx="200" cy="100" r="80" fill-opacity="0.2" stroke="black" stroke-width="2" fill="red" />
  <circle cx="100" cy="100" r="80" fill-opacity="0.2" stroke="black" stroke-width="2" fill="green" />
  <line x1="150" y1="125" x2="300" y2="150" stroke="black" stroke-width="2" fill="black" stroke-dasharray="5,3"/>
  <text x="300" y="165" font-family="Verdana" font-size="35">Everything but this!</text>
</svg>
```


<svg width="800" height="300">
  <circle cx="150" cy="180" r="80" fill-opacity="0.2" stroke="black" stroke-width="2" fill="blue" />
  <circle cx="200" cy="100" r="80" fill-opacity="0.2" stroke="black" stroke-width="2" fill="red" />
  <circle cx="100" cy="100" r="80" fill-opacity="0.2" stroke="black" stroke-width="2" fill="green" />
  <line x1="150" y1="125" x2="300" y2="150" stroke="black" stroke-width="2" fill="black" stroke-dasharray="5,3"/>
  <text x="300" y="165" font-family="Verdana" font-size="35">Everything but this!</text>
</svg>




```python
def answer_two():
    # YOUR CODE HERE
    global Energy, GDP, ScimEn
    All = pd.merge(pd.merge(Energy, GDP, on='Country', how='outer'), ScimEn, on='Country', how='outer')
    return (len(All)-len(answer_one()))
    raise NotImplementedError()
```


```python
assert type(answer_two()) == int, "Q2: You should return an int number!"

```

### Question 3
What are the top 15 countries for average GDP over the last 10 years?

*This function should return a Series named `avgGDP` with 15 countries and their average GDP sorted in descending order.*


```python
def answer_three():
    # YOUR CODE HERE
    df = answer_one()
    avgGDP = df[['2006', '2007', '2008', '2009', '2010', '2011', '2012',
                    '2013', '2014', '2015']].mean(axis=1).sort_values(ascending=False)
    return avgGDP
    raise NotImplementedError()
```




    Country
    United States         1.536434e+13
    China                 6.348609e+12
    Japan                 5.542208e+12
    Germany               3.493025e+12
    France                2.681725e+12
    United Kingdom        2.487907e+12
    Brazil                2.189794e+12
    Italy                 2.120175e+12
    India                 1.769297e+12
    Canada                1.660647e+12
    Russian Federation    1.565459e+12
    Spain                 1.418078e+12
    Australia             1.164043e+12
    South Korea           1.106715e+12
    Iran                  4.441558e+11
    dtype: float64




```python
assert type(answer_three()) == pd.Series, "Q3: You should return a Series!"

```

### Question 4
By how much had the GDP changed over the 10 year span for the country with the 6th largest average GDP?

*This function should return a single number.*


```python
def answer_four():
    # YOUR CODE HERE
    df = answer_one()
    avgGDP = answer_three().reset_index()
    return np.float64(df.filter(like=avgGDP.iloc[5]['Country'], axis=0)['2015'] - df.filter(like=avgGDP.iloc[5]['Country'], axis=0)['2006'])
    raise NotImplementedError()
```


```python
# Cell for autograder.

```

### Question 5
What is the mean energy supply per capita?

*This function should return a single number.*


```python
def answer_five():
    # YOUR CODE HERE
    df = answer_one()['Energy Supply per Capita']
    return df.mean()
    raise NotImplementedError()
```


```python
# Cell for autograder.

```

### Question 6
What country has the maximum % Renewable and what is the percentage?

*This function should return a tuple with the name of the country and the percentage.*


```python
def answer_six():
    # YOUR CODE HERE
    df = answer_one()
    country = df.sort_values(by='% Renewable', ascending=False).iloc[0]
    return (country.name, country['% Renewable'])
    raise NotImplementedError()
```


```python
assert type(answer_six()) == tuple, "Q6: You should return a tuple!"

assert type(answer_six()[0]) == str, "Q6: The first element in your result should be the name of the country!"

```

### Question 7
Create a new column that is the ratio of Self-Citations to Total Citations. 
What is the maximum value for this new column, and what country has the highest ratio?

*This function should return a tuple with the name of the country and the ratio.*


```python
def answer_seven():
    # YOUR CODE HERE
    citations = answer_one()[['Self-citations', 'Citations']]
    citations['Ratio']= citations['Self-citations']/citations['Citations']
    return ( citations['Ratio'].idxmax(), citations['Ratio'].max())
    raise NotImplementedError()
```


```python
assert type(answer_seven()) == tuple, "Q7: You should return a tuple!"

assert type(answer_seven()[0]) == str, "Q7: The first element in your result should be the name of the country!"

```

### Question 8

Create a column that estimates the population using Energy Supply and Energy Supply per capita. 
What is the third most populous country according to this estimate?

*This function should return the name of the country*


```python
def answer_eight():
    # YOUR CODE HERE
    consum = answer_one()[['Energy Supply','Energy Supply per Capita']]
    consum['population'] = consum['Energy Supply']/consum['Energy Supply per Capita']
    return consum['population'].sort_values().index[2]
    raise NotImplementedError()
```


```python
assert type(answer_eight()) == str, "Q8: You should return the name of the country!"

```

### Question 9
Create a column that estimates the number of citable documents per person. 
What is the correlation between the number of citable documents per capita and the energy supply per capita? Use the `.corr()` method, (Pearson's correlation).

*This function should return a single number.*

*(Optional: Use the built-in function `plot9()` to visualize the relationship between Energy Supply per Capita vs. Citable docs per Capita)*


```python
def answer_nine():
    # YOUR CODE HERE
    All_df = answer_one()
    All_df['population'] = All_df['Energy Supply']/All_df['Energy Supply per Capita']
    All_df['CitePercap'] = All_df['Citable documents']/All_df['population']
    All_df['CitePercap']=np.float64(All_df['CitePercap'])
    cor = All_df[['Energy Supply per Capita','CitePercap']].corr().iloc[0,1]
    return cor
    raise NotImplementedError()
```


```python
def plot9():
    import matplotlib as plt
    %matplotlib inline
    
    Top15 = answer_one()
    Top15['PopEst'] = Top15['Energy Supply'] / Top15['Energy Supply per Capita']
    Top15['Citable docs per Capita'] = Top15['Citable documents'] / Top15['PopEst']
    Top15.plot(x='Citable docs per Capita', y='Energy Supply per Capita', kind='scatter', xlim=[0, 0.0006])
```


```python
assert answer_nine() >= -1. and answer_nine() <= 1., "Q9: A valid correlation should between -1 to 1!"

```

### Question 10
Create a new column with a 1 if the country's % Renewable value is at or above the median for all countries in the top 15, and a 0 if the country's % Renewable value is below the median.

*This function should return a series named `HighRenew` whose index is the country name sorted in ascending order of rank.*


```python
def answer_ten():
    # YOUR CODE HERE
    df = answer_one()
    df['HighRenew'] = df['% Renewable'] >= df['% Renewable'].median()
    df['HighRenew'] = df['HighRenew'].apply(lambda x: 1 if x else 0)
    return df['HighRenew']
    raise NotImplementedError()
```


```python
assert type(answer_ten()) == pd.Series, "Q10: You should return a Series!"

```

### Question 11
Use the following dictionary to group the Countries by Continent, then create a DataFrame that displays the sample size (the number of countries in each continent bin), and the sum, mean, and std deviation for the estimated population of each country.

```python
ContinentDict  = {'China':'Asia', 
                  'United States':'North America', 
                  'Japan':'Asia', 
                  'United Kingdom':'Europe', 
                  'Russian Federation':'Europe', 
                  'Canada':'North America', 
                  'Germany':'Europe', 
                  'India':'Asia',
                  'France':'Europe', 
                  'South Korea':'Asia', 
                  'Italy':'Europe', 
                  'Spain':'Europe', 
                  'Iran':'Asia',
                  'Australia':'Australia', 
                  'Brazil':'South America'}
```

*This function should return a DataFrame with index named Continent `['Asia', 'Australia', 'Europe', 'North America', 'South America']` and columns `['size', 'sum', 'mean', 'std']`*


```python
def answer_eleven():
    # YOUR CODE HERE
    ContinentDict  = {'China':'Asia', 
                  'United States':'North America', 
                  'Japan':'Asia', 
                  'United Kingdom':'Europe', 
                  'Russian Federation':'Europe', 
                  'Canada':'North America', 
                  'Germany':'Europe', 
                  'India':'Asia',
                  'France':'Europe', 
                  'South Korea':'Asia', 
                  'Italy':'Europe', 
                  'Spain':'Europe', 
                  'Iran':'Asia',
                  'Australia':'Australia', 
                  'Brazil':'South America'}
    summaries = pd.DataFrame(columns = ['size', 'sum', 'mean', 'std'])
    df = answer_one()
    
    df['Estimate Population'] = df['Energy Supply'] / df['Energy Supply per Capita']
    for group, frame in df.groupby(ContinentDict):
        summaries.loc[group] = [len(frame), frame['Estimate Population'].sum(),frame['Estimate Population'].mean(),frame['Estimate Population'].std()]
    return summaries
    raise NotImplementedError()
```


```python
assert type(answer_eleven()) == pd.DataFrame, "Q11: You should return a DataFrame!"

assert answer_eleven().shape[0] == 5, "Q11: Wrong row numbers!"

assert answer_eleven().shape[1] == 4, "Q11: Wrong column numbers!"

```

### Question 12
Cut % Renewable into 5 bins. Group Top15 by the Continent, as well as these new % Renewable bins. How many countries are in each of these groups?

*This function should return a Series with a MultiIndex of `Continent`, then the bins for `% Renewable`. Do not include groups with no countries.*


```python
def answer_twelve():
    # YOUR CODE HERE
    ContinentDict  = {'China':'Asia', 
                  'United States':'North America', 
                  'Japan':'Asia', 
                  'United Kingdom':'Europe', 
                  'Russian Federation':'Europe', 
                  'Canada':'North America', 
                  'Germany':'Europe', 
                  'India':'Asia',
                  'France':'Europe', 
                  'South Korea':'Asia', 
                  'Italy':'Europe', 
                  'Spain':'Europe', 
                  'Iran':'Asia',
                  'Australia':'Australia', 
                  'Brazil':'South America'}
    df = answer_one()
    df["bins"] = pd.cut(df["% Renewable"], 5)
    val = df.groupby([ContinentDict, df['bins']]).size()
    return val
    raise NotImplementedError()
```




    Country        bins            
    Asia           (2.212, 15.753]     4
                   (15.753, 29.227]    1
                   (29.227, 42.701]    0
                   (42.701, 56.174]    0
                   (56.174, 69.648]    0
    Australia      (2.212, 15.753]     1
                   (15.753, 29.227]    0
                   (29.227, 42.701]    0
                   (42.701, 56.174]    0
                   (56.174, 69.648]    0
    Europe         (2.212, 15.753]     1
                   (15.753, 29.227]    3
                   (29.227, 42.701]    2
                   (42.701, 56.174]    0
                   (56.174, 69.648]    0
    North America  (2.212, 15.753]     1
                   (15.753, 29.227]    0
                   (29.227, 42.701]    0
                   (42.701, 56.174]    0
                   (56.174, 69.648]    1
    South America  (2.212, 15.753]     0
                   (15.753, 29.227]    0
                   (29.227, 42.701]    0
                   (42.701, 56.174]    0
                   (56.174, 69.648]    1
    dtype: int64




```python
assert type(answer_twelve()) == pd.Series, "Q12: You should return a Series!"

assert len(answer_twelve()) == 25, "Q12: Wrong result numbers!"

```

### Question 13
Convert the Population Estimate series to a string with thousands separator (using commas). Use all significant digits (do not round the results).

e.g. 12345678.90 -> 12,345,678.90

*This function should return a series `PopEst` whose index is the country name and whose values are the population estimate string*


```python
def answer_thirteen():
    # YOUR CODE HERE
    df = answer_one()
    df["PopEst"] = df['Energy Supply'] / df['Energy Supply per Capita']
    return df['PopEst'].apply(lambda x: '{0:,}'.format(x))
    raise NotImplementedError()
```


```python
assert type(answer_thirteen()) == pd.Series, "Q13: You should return a Series!"

assert len(answer_thirteen()) == 15, "Q13: Wrong result numbers!"

```

### Optional

Use the built in function `plot_optional()` to see an example visualization.


```python
def plot_optional():
    import matplotlib as plt
    %matplotlib inline
    Top15 = answer_one()
    ax = Top15.plot(x='Rank', y='% Renewable', kind='scatter', 
                    c=['#e41a1c','#377eb8','#e41a1c','#4daf4a','#4daf4a','#377eb8','#4daf4a','#e41a1c',
                       '#4daf4a','#e41a1c','#4daf4a','#4daf4a','#e41a1c','#dede00','#ff7f00'], 
                    xticks=range(1,16), s=6*Top15['2014']/10**10, alpha=.75, figsize=[16,6]);

    for i, txt in enumerate(Top15.index):
        ax.annotate(txt, [Top15['Rank'][i], Top15['% Renewable'][i]], ha='center')

    print("This is an example of a visualization that can be created to help understand the data. \
This is a bubble chart showing % Renewable vs. Rank. The size of the bubble corresponds to the countries' \
2014 GDP, and the color corresponds to the continent.")
```


```python

```
