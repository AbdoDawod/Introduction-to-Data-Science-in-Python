# Assignment 4
## Description
In this assignment you must read in a file of metropolitan regions and associated sports teams from [assets/wikipedia_data.html](assets/wikipedia_data.html) and answer some questions about each metropolitan region. Each of these regions may have one or more teams from the "Big 4": NFL (football, in [assets/nfl.csv](assets/nfl.csv)), MLB (baseball, in [assets/mlb.csv](assets/mlb.csv)), NBA (basketball, in [assets/nba.csv](assets/nba.csv) or NHL (hockey, in [assets/nhl.csv](assets/nhl.csv)). Please keep in mind that all questions are from the perspective of the metropolitan region, and that this file is the "source of authority" for the location of a given sports team. Thus teams which are commonly known by a different area (e.g. "Oakland Raiders") need to be mapped into the metropolitan region given (e.g. San Francisco Bay Area). This will require some human data understanding outside of the data you've been given (e.g. you will have to hand-code some names, and might need to google to find out where teams are)!

For each sport I would like you to answer the question: **what is the win/loss ratio's correlation with the population of the city it is in?** Win/Loss ratio refers to the number of wins over the number of wins plus the number of losses. Remember that to calculate the correlation with [`pearsonr`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.pearsonr.html), so you are going to send in two ordered lists of values, the populations from the wikipedia_data.html file and the win/loss ratio for a given sport in the same order. Average the win/loss ratios for those cities which have multiple teams of a single sport. Each sport is worth an equal amount in this assignment (20%\*4=80%) of the grade for this assignment. You should only use data **from year 2018** for your analysis -- this is important!

## Notes

1. Do not include data about the MLS or CFL in any of the work you are doing, we're only interested in the Big 4 in this assignment.
2. I highly suggest that you first tackle the four correlation questions in order, as they are all similar and worth the majority of grades for this assignment. This is by design!
3. It's fair game to talk with peers about high level strategy as well as the relationship between metropolitan areas and sports teams. However, do not post code solving aspects of the assignment (including such as dictionaries mapping areas to teams, or regexes which will clean up names).
4. There may be more teams than the assert statements test, remember to collapse multiple teams in one city into a single value!

## Question 1
For this question, calculate the win/loss ratio's correlation with the population of the city it is in for the **NHL** using **2018** data.


```python
%%capture
#RUN FIRST, installs a missing library
import sys
!{sys.executable} -m pip install lxml==4.4.1
```


```python
import pandas as pd
import numpy as np
import scipy.stats as stats
import re

def nhl_df_cleaned():
    nhl_df=pd.read_csv("assets/nhl.csv")
    cities=pd.read_html("assets/wikipedia_data.html")[1]
    cities=cities.iloc[:-1,[0,3,5,6,7,8]]
    nhl_df = nhl_df[nhl_df['year']==2018]
    nhl_df = nhl_df[~nhl_df['team'].isin(['Atlantic Division','Metropolitan Division' , 'Central Division', 'Pacific Division'])]
    nhl_df['team'] = nhl_df['team'].apply(lambda x: re.sub('\*', '', x))
    nhl_df["Team"] = nhl_df["team"].apply(lambda x: x.split(" ")[-1])

    cities['NHL']=cities['NHL'].apply(lambda x: re.sub('\[.*?\]', '', x))
    cities["NHL"] = cities["NHL"].replace({"Rangers Islanders Devils": "Rangers,Islanders,Devils",
                                           "Kings Ducks": "Kings,Ducks",
                                           "Red Wings": "Red,Wings", 
                                           "Maple Leafs": "Maple,Leafs", 
                                           "Blue Jackets": "Blue,Jackets",
                                           "Golden Knights": "Golden,Knights" })
    cities["NHL"] = cities["NHL"].apply(lambda x: x.split(","))
    cities = cities.explode("NHL")
#
    df2 = pd.merge(cities, nhl_df, left_on="NHL", right_on="Team").drop(columns=['team'])[["Metropolitan area", "Population (2016 est.)[8]", "NHL", "Team", "W", "L"]]
    df2['W/L'] = df2['W'].astype(int)/(df2['W'].astype(int)+df2['L'].astype(int))
    
    df2.loc[df2["Metropolitan area"] == "New York City", "W/L"] = 0.5182013333333334 # mean of NY W-L%
    df2.loc[df2["Metropolitan area"] == "Los Angeles", "W/L"] = 0.6228945 # mean of LA W-L%
    df2 = df2.drop_duplicates(subset="Metropolitan area").reset_index()
    df2 = df2.drop(columns="index")
    return df2

def nhl_correlation(): 
    # Load data
    df2 = nhl_df_cleaned()
    #raise NotImplementedError()
    
    population_by_region = df2['Population (2016 est.)[8]'].astype(float) # pass in metropolitan area population from cities
    win_loss_by_region = df2['W/L'].astype(float) # pass in win/loss ratio from nhl_df in the same order as cities["Metropolitan area"]

    assert len(population_by_region) == len(win_loss_by_region), "Q1: Your lists must be the same length"
    assert len(population_by_region) == 28, "Q1: There should be 28 teams being analysed for NHL"
    
    return stats.pearsonr(population_by_region, win_loss_by_region)[0]
```




    (0.012485959345532897, 0.9497191047673461)




```python

```

## Question 2
For this question, calculate the win/loss ratio's correlation with the population of the city it is in for the **NBA** using **2018** data.


```python
import pandas as pd
import numpy as np
import scipy.stats as stats
import re

def nba_df_cleaned():
    nba_df=pd.read_csv("assets/nba.csv")
    cities=pd.read_html("assets/wikipedia_data.html")[1]
    cities=cities.iloc[:-1,[0,3,5,6,7,8]]
    NBA_df = nba_df[nba_df['year']==2018]
    NBA_df['team']=NBA_df['team'].apply(lambda x: re.sub(r"(\*)*\s\(\d+\)", "", x))
    NBA_df['team'] = NBA_df['team'].apply(lambda x: re.sub('\*', '', x))
    NBA_df["team"] = NBA_df["team"].apply(lambda x: x.split(" ")[-1])
    NBA_df['W/L%'] = pd.to_numeric(NBA_df['W/L%'], errors='coerce')
    NBA_df.dropna(subset=['W/L%'], inplace=True)

    cities['NBA']=cities['NBA'].apply(lambda x: re.sub('\[.*?\]', '', x))
    cities["NBA"] = cities["NBA"].replace({"Knicks Nets": "Knicks,Nets",
                                           "Lakers Clippers": "Lakers,Clippers",
                                           "Trail Blazers": "Trail,Blazers"})
    cities["NBA"] = cities["NBA"].apply(lambda x: x.split(","))
    cities = cities.explode("NBA")
#
    df2 = pd.merge(cities, NBA_df, left_on="NBA", right_on="team")[["Metropolitan area", "Population (2016 est.)[8]", "NBA", "team", "W/L%"]]

    df2.loc[df2["Metropolitan area"] == "New York City", "W/L%"] = 0.34750000000000003 # mean of NY W-L%
    df2.loc[df2["Metropolitan area"] == "Los Angeles", "W/L%"] = 0.46950000000000003 # mean of LA W-L%
    df2 = df2.drop_duplicates(subset="Metropolitan area").reset_index()
    df2 = df2.drop(columns="index") 
    return df2

def nba_correlation():

    df2 = nba_df_cleaned()
    #raise NotImplementedError()
    
    population_by_region = df2['Population (2016 est.)[8]'].astype(float) # pass in metropolitan area population from cities
    win_loss_by_region = df2['W/L%'].astype(float) # pass in win/loss ratio from nba_df in the same order as cities["Metropolitan area"]
    assert len(population_by_region) == len(win_loss_by_region), "Q2: Your lists must be the same length"
    assert len(population_by_region) == 28, "Q2: There should be 28 teams being analysed for NBA"

    return stats.pearsonr(population_by_region, win_loss_by_region)[0]
```




    (-0.17636350642182935, 0.36932106185547353)




```python

```

## Question 3
For this question, calculate the win/loss ratio's correlation with the population of the city it is in for the **MLB** using **2018** data.


```python
import pandas as pd
import numpy as np
import scipy.stats as stats
import re

def mlb_df_cleaned():
    mlb_df=pd.read_csv("assets/mlb.csv")
    cities=pd.read_html("assets/wikipedia_data.html")[1]
    cities=cities.iloc[:-1,[0,3,5,6,7,8]]
    mlb_df = mlb_df[mlb_df['year']==2018]
    mlb_df["team"] = mlb_df["team"].apply(lambda x: x.split(" ")[-1])

    cities['MLB']=cities['MLB'].apply(lambda x: re.sub('\[.*?\]', '', x))
    cities["MLB"] = cities["MLB"].replace({"Blue Jays": "Blue,Jays", 
                                           "Cubs White Sox": "Cubs,White,Sox", 
                                           "Dodgers Angels": "Dodgers,Angels", 
                                           "Giants Athletics": "Giants,Athletics", 
                                           "Yankees Mets": "Yankees,Mets",
                                           "Red Sox": "Red,Sox"})
    cities["MLB"] = cities["MLB"].apply(lambda x: x.split(","))
    cities = cities.explode("MLB")
    df2 = pd.merge(cities, mlb_df, left_on="MLB", right_on="team")[["Metropolitan area", "Population (2016 est.)[8]", "MLB", "team", "W-L%"]]
    
    df2.loc[df2["Metropolitan area"] == "New York City", "W-L%"] = 0.546 # mean of NY W-L%
    df2.loc[df2["Metropolitan area"] == "Los Angeles", "W-L%"] = 0.5289999999999999 # mean of LA W-L%
    df2.loc[df2["Metropolitan area"] == "San Francisco Bay Area", "W-L%"] = 0.525 # mean of SF W-L%
    df2.loc[df2["Metropolitan area"] == "Chicago", "W-L%"] = 0.482769 # mean of CH W-L%
    df2.loc[df2["Metropolitan area"] == "Boston", "W-L%"] = 0.666667 # mean of BO W-L%
    df2 = df2.drop_duplicates(subset="Metropolitan area").reset_index()
    df2 = df2.drop(columns="index")
    return df2
    

def mlb_correlation(): 
    df2 = mlb_df_cleaned()
    #raise NotImplementedError()

    population_by_region = df2['Population (2016 est.)[8]'].astype(float)  # pass in metropolitan area population from cities
    win_loss_by_region = df2['W-L%'].astype(float) # pass in win/loss ratio from mlb_df in the same order as cities["Metropolitan area"]
    
    assert len(population_by_region) == len(win_loss_by_region), "Q3: Your lists must be the same length"
    assert len(population_by_region) == 26, "Q3: There should be 26 teams being analysed for MLB"

    return stats.pearsonr(population_by_region, win_loss_by_region)[0]
```




    (0.14999221533065174, 0.46456427053982524)




```python

```

## Question 4
For this question, calculate the win/loss ratio's correlation with the population of the city it is in for the **NFL** using **2018** data.


```python
import pandas as pd
import numpy as np
import scipy.stats as stats
import re

def nfl_df_cleaned():
    nfl_df=pd.read_csv("assets/nfl.csv")
    cities=pd.read_html("assets/wikipedia_data.html")[1]
    cities=cities.iloc[:-1,[0,3,5,6,7,8]]
    nfl_df['W-L%'] = pd.to_numeric(nfl_df['W-L%'], errors='coerce')
    nfl_df.dropna(subset=['W-L%'], inplace=True)
    nfl_df = nfl_df[nfl_df['year']==2018]
    nfl_df["team"] = nfl_df["team"].apply(lambda x: x.split(" ")[-1])
    nfl_df['team'] = nfl_df['team'].apply(lambda x: re.sub('[\*\+]', '', x))

    cities["NFL"] = cities["NFL"].apply(lambda x: re.sub(r"\[.+\]", "", x))
    cities["NFL"] = cities["NFL"].replace({"Giants Jets": "Giants,Jets",
                                           "Rams Chargers": "Rams,Chargers",
                                           "49ers Raiders": "49ers,Raiders"
                                           })
    cities["NFL"] = cities["NFL"].apply(lambda x: x.split(","))
    cities = cities.explode("NFL")

    df2 = pd.merge(cities, nfl_df, left_on="NFL", right_on="team")[["Metropolitan area", "Population (2016 est.)[8]", "NFL", "W-L%"]]
    df2.loc[df2["Metropolitan area"] == "New York City", "W-L%"] = 0.2815 # mean of NY W-L%
    df2.loc[df2["Metropolitan area"] == "Los Angeles", "W-L%"] = 0.7815 # mean of LA W-L%
    df2.loc[df2["Metropolitan area"] == "San Francisco Bay Area", "W-L%"] = 0.25 # mean of SF W-L%
    df2 = df2.drop_duplicates(subset="Metropolitan area").reset_index()
    df2 = df2.drop(columns="index")
    return df2

def nfl_correlation(): 
    df2 = nfl_df_cleaned()
    #raise NotImplementedError()
    
    population_by_region = df2["Population (2016 est.)[8]"].astype(float) # pass in metropolitan area population from cities
    win_loss_by_region = df2["W-L%"].astype(float) # pass in win/loss ratio from nfl_df in the same order as cities["Metropolitan area"]

    assert len(population_by_region) == len(win_loss_by_region), "Q4: Your lists must be the same length"
    assert len(population_by_region) == 29, "Q4: There should be 29 teams being analysed for NFL"

    return stats.pearsonr(population_by_region, win_loss_by_region)[0]
```




    (0.0042821414363930195, 0.9824114740736553)




```python

```

## Question 5
In this question I would like you to explore the hypothesis that **given that an area has two sports teams in different sports, those teams will perform the same within their respective sports**. How I would like to see this explored is with a series of paired t-tests (so use [`ttest_rel`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.ttest_rel.html)) between all pairs of sports. Are there any sports where we can reject the null hypothesis? Again, average values where a sport has multiple teams in one region. Remember, you will only be including, for each sport, cities which have teams engaged in that sport, drop others as appropriate. This question is worth 20% of the grade for this assignment.


```python
import pandas as pd
import numpy as np
import scipy.stats as stats
import re

mlb_df=pd.read_csv("assets/mlb.csv")
nhl_df=pd.read_csv("assets/nhl.csv")
nba_df=pd.read_csv("assets/nba.csv")
nfl_df=pd.read_csv("assets/nfl.csv")
cities=pd.read_html("assets/wikipedia_data.html")[1]
cities=cities.iloc[:-1,[0,3,5,6,7,8]]

def sports_team_performance():
    # YOUR CODE HERE
    NFL_df = nfl_df_cleaned()[["Metropolitan area", "W-L%"]]
    NBA_df = nba_df_cleaned().rename(columns={'W/L%': 'W-L%'})[["Metropolitan area","W-L%" ]]
    NHL_df = nhl_df_cleaned().rename(columns={'W/L': 'W-L%'})[["Metropolitan area","W-L%" ]] 
    MLB_df = mlb_df_cleaned()[["Metropolitan area", "W-L%"]]
    
    leagues = {"NFL": NFL_df, "NBA": NBA_df, "NHL": NHL_df, "MLB": MLB_df}
    p_values = pd.DataFrame(columns=leagues.keys(), index=leagues.keys())
    for (league_name1, league_df1) in leagues.items():
        for (league_name2, league_df2) in leagues.items():
            merged_league = pd.merge(league_df1, league_df2, on="Metropolitan area")
            p_values.loc[league_name1, league_name2] = stats.ttest_rel(merged_league["W-L%_x"], merged_league["W-L%_y"])[1]
            
   # p_values_dict = calculate_p_values(leagues)
    p_values = pd.DataFrame(p_values).astype("float")
    #raise NotImplementedError()
    
    # Note: p_values is a full dataframe, so df.loc["NFL","NBA"] should be the same as df.loc["NBA","NFL"] and
    # df.loc["NFL","NFL"] should return np.nan
    sports = ['NFL', 'NBA', 'NHL', 'MLB']
    p_values = pd.DataFrame({k:np.nan for k in sports}, index=sports)
    
    assert abs(p_values.loc["NBA", "NHL"] - 0.02) <= 1e-2, "The NBA-NHL p-value should be around 0.02"
    assert abs(p_values.loc["MLB", "NFL"] - 0.80) <= 1e-2, "The MLB-NFL p-value should be around 0.80"
    return p_values
```


```python

```
