# League of Legend Victory Analysis

## Introduction

League of Legends is a free-to-play multiplayer game where teams of 5 players each battle each other in the place called summoners rift. Summoners Rift is where I have spent too much time as a kid. Playing the game, it's natural to try to find the best way to win as the game is incredibly technical, requires great degrees of teamwork, and roughly an hour free to play a single match. It can be a long game to play and one would want to make sure their time is worth it. So this raises the big question we will attempt to explore today

### What game objectives in League of Legends are more likely to be taken by winning teams in pro-play matches in 2022?

To work towards answering this question, we will use a dataset provided by https://oracleselixir.com/. This dataset contains the match history of as many pro leagues as possible during the 2022 pro-play season. The raw dataset contains roughly 149232 rows and 123 columns. Luckily for us and my laptop's 12 GB of RAM, we will only care for a few columns as shown below:


| Column | Type | Description|
| ---------------- | ---------| --------------------- |
| 'gameid'| Str | Unique id for each proplay match|
| 'teamid'| Str | Unique id for each team in the dataset. This will be helpful for seeing overall team performance |
| 'Playername' | Str | Name of each player. If the name is not known, it's unknown_player but there are still nan values. This will come into play during data cleaning. |
| 'url' | Object | Depending on the source the match data came from, the dataset may contain a link to the site where the data was pulled from. We will discuss more this when discussing potential issues with missingness in our dataset.
| League | Str | Not to be confused with league of legends, the game title, league refers to the gaming league the pro match was played under. Again, this will help us discuss potential missingness in our dataset |
| playoffs | Bool | Was the game a playoff match? |
| gamelength | int | the length in seconds of a match |
| firstblood | Bool | Which team got first blood in a match? A player getting frist blood means that the player was the first in a match to kill another player of the opposing team. Perhaps first blood could create early leads? | 
| firstdragon | Bool | Which team got the first dragon in a match? Killing a dragon grants the team extra powers which could give early advantages to teams. Wonder if it pays off. |
| firstherald | Bool | Which team got the first Heralds, bosses that when a team kills them, joins said team and takes down defenses of the opposing team. Perhaps winning teams like using them? |
| firstbaron | Bool | Which team got the first Baron, a massive boss that provides a huge power-up to a team that gets it late in the match? Will definitely be worth seeing what kind of advantage a baron could give |
| firsttower | Bool | Which team gets the first tower? Towers are a team's defenses, breaking them down gives an opposing team easier access to a team's nexus and victory. How often do firsttowers come into play? |
| firstmidtower | Bool | Which team gets the first midlane tower? The midlane is special in particular becuase it is the shortest path to the other team's nexus. Could bringing it down create an advantage? |
| firstmidtower | Bool | which team gets 3 towers frist? See firsttower and fristmidtower above for more details about towers |
| csat15 | Int | How much CS (critter score, number of monsters killed) a team got by 15 minutes into a match. The original dataset had a few of these kinds of statistics |
| kills | Int | how many times has the team killed a member of the opposing team |

# Data Cleaning and EDA

## Data Cleaning
Before getting into any analysis, first, we need to discuss data cleaning. The dataset itself aggregates data from serval different sources ("Match History pages, lolesports.com, lpl.QQ.com, Leaguepedia, the Riot Games solo queue APIs, and more" - https://oracleselixir.com/faq). So the downloaded dataset has a few bits we need to clean up to make analysis easier. 

First and foremost, the dataset contains data on both overall team performance and individual player performance. Today we care about overall team performance. Since both team player and individual player performance are in the same CSV file, we need to find a way to spilt that up and get the team measurements. For that, we can look at the playername/playerid column of the dataset. If we group playername by gameid, count the number of players in each group then compute how often each value appears in that count, we observe that 10 appears 12436 times. Since we know 10 players play in any given game, then this implies there are 12436 games in the database. Now if we want to find data on each team in the dataset, and we know that there are two teams in each match, then we should expect to find some data point that appears 12436 * 2 times. The guidance here is matches don't logically have playernames! Sure enough, counting the number of null items in the playername column add up to 12436 * 2. So our playerdata is playerdata if playername is non null and our team data is when playername is null. 

Once that more complicated data cleaning is dealt with, we can do more standard data cleaning! The first note is that many columns needed for the analysis are integers when they can be booleans. The result of the match is 1 for a win and 0 for a loss. Why not switch that for True if a team wins and false if they lose? This was likewise done for Playoff, firstblood, firstdragon,firstherald,firsttower,firstmidtower, and firstmidtower.

This allowed us to create the dataframe, as shown below:


'|    | gameid                | teamid                                  | league   | url                                         | playoffs   | firstblood   | firstdragon   | firstherald   | firstbaron   | firsttower   | firstmidtower   | firstmidtower   |   csat15 |   kills |\n|---:|:----------------------|:----------------------------------------|:---------|:--------------------------------------------|:-----------|:-------------|:--------------|:--------------|:-------------|:-------------|:----------------|:----------------|---------:|--------:|\n|  0 | ESPORTSTMNT01_2690210 | oe:team:68911b3329146587617ab2973106e23 | LCK CL   | nan                                         | False      | True         | False         | True          | False        | True         | True            | True            |      487 |       9 |\n|  1 | ESPORTSTMNT01_2690210 | oe:team:d2dc3681437e2beb2bb4742477108ff | LCK CL   | nan                                         | False      | False        | True          | False         | False        | False        | False           | False           |      510 |      19 |\n|  2 | ESPORTSTMNT01_2690219 | oe:team:6dcacec00a6ba7576c5ab7f30c995cd | LCK CL   | nan                                         | False      | False        | False         | True          | False        | False        | False           | False           |      533 |       3 |\n|  3 | ESPORTSTMNT01_2690219 | oe:team:5380cdbc2ad2b8082624f48f99f6672 | LCK CL   | nan                                         | False      | True         | True          | False         | True         | True         | True            | True            |      555 |      16 |\n|  4 | 8401-8401_game_1      | oe:team:f4c4528c6981e104a11ea7548630c23 | LPL      | https://lpl.qq.com/es/stats.shtml?bmid=8401 | False      | False        | True          | True          | True         | True         | True            | True            |      nan |      13 |'


## Univariate Analysis

The next step is Exploratory Data Analysis and looking at how the data stacks up. Interestingly enough, many of our numeric columns were mostly unimodal and normally distributed as seen below with this plot the number of minions/monsters (cs) killed total by the team 10 minutes into the game. Here the plot is normally distributed with most csat10 done between 250 to 350 CS total (so about 50 to 70 cs for each player in a team of 5).  

<iframe src="assets/csat15.html" width=800 height=600 frameBorder=0></iframe> 

However, a handful of columns were also bimodal, as was the case with the number of kills in team games. Notice that there is a peak at roughly 10 and 20 kills in a game.

<iframe src="assets/kills_a.html" width=800 height=600 frameBorder=0></iframe>

I wondered if the difference between these graphs could have something to do with how winning games impact play. For instance, do losing teams not have CS affected by a loss, or do winning teams have more kills? Do winning and losing change the distribution of certain stats? 

## Looking Closer at the Univariate Analysis 

Out of curiosity sake, I took our csat15 column and our kills col, spilt both into two different arrays based on if the team won or lose, then graphed both arrays. Starting with the csat15, we see that, either win or lose, CS is roughly the same.

<iframe src="assets/csat15_b.html" width=800 height=600 frameBorder=0></iframe> 

Then as predicted with kills, we see that kills definitely have a different distribution depending on if a team won or lost.

<iframe src="assets/kills_b.html" width=800 height=600 frameBorder=0></iframe> 

So it would seem that the bimodality of the graph has a possible relationship with if a team won or lost as aggregating those different distributions together form a more bimodal normal graph. 

## Bivariate Analysis

Upon looking at a few scatter plots, this particular scatter plot stands out. Here we have a point for each team. Each point has its average win_rate in the dataset and the average time they take the first baron. Here there appears to be a positive relationship between a team's winrate and how often the team takes the first baron in their games! We will explore this relationship further in the hypothesis test. 

<iframe src="assets/firstbaron.html" width=800 height=600 frameBorder=0></iframe> 

## Interesting Aggregates

One interesting groupby aggregate worth mentioning is the one below. Here we ran the following code snippet.

```
col = lol_team_data.columns.str.contains("first") | lol_team_data.columns.isin(["teamid","result"])
lol_team_data.loc[:,col].groupby("result").mean()
```

What this code does is take all the "first" columns for each team then groups the result to find, on average, how often they occur alongside a winning team or a losing team. Below we can observe that consistently on average, winning teams usually are the ones to obtain some objective before the other team. Note that the highest average for a winning team to take some objective is for fristbaron. It would seem to point more to a relationship between winning and taking fristbaron. 


' result   |   firstblood |   firstdragon |   firstherald |   firstbaron |   firsttower |   firstmidtower |   firsttothreetowers |\n|:---------|-------------:|--------------:|--------------:|-------------:|-------------:|----------------:|---------------------:|\n| False    |     0.390738 |      0.507638 |      0.502653 |     0.267969 |     0.416546 |        0.378839 |             0.335665 |\n| True     |     0.607849 |      0.638491 |      0.643397 |     0.833601 |     0.729773 |        0.767653 |             0.81052  |'


# Assessment of Missingness


## NMAR Missingness

I am inclined to think there is not a NMAR column in our dataset primarily for 3 possible reasons: the league the match took place in, if the match was a playoff, the game_length.

Suppose we believe we didn't have a column to describe the missingness of our columns, thereby making a few of the columns NMAR. These columns' missingness from the data-generating process arises when the data associated with that match is not present in any of the data collection sites the dataset was scraped. The big question then becomes why those data points are not present in the data collection sites which brings us to the different leagues. Each league is going to have different policies on how much data about the match is circulated or each league has different data collection agencies observing the match and collecting data on it. So if we wanted to determine the missingness of a certain column, it makes the most sense to know what league the match took place in so we can determine what data collection policies are in place. We do have information about the leagues so our missingness can depend on the leagues themselves.

This logic is similar to playoffs. Playoffs may be more expensive productions and many more watchful eyes taking note of data coming from the competition. Not to mention such a high-level competition would have people watching to see how the best performance and alongside that we would get more data than a match that is not in a playoff potentially.

Finally, there is gamelength. If a game breaks early or there is an early surrender, we may miss out on some of the data points that are tied to the length of the game, such as csat10. If the game ended early by random chance, we would not be able to measure csat10 or other similar statistics. Hence those missingness would depend on how long the game actually progressed.

The long and short of it is, the original dataset contains over 100 columns. The rule of thumb for NMAR is the more data you have, the less likely the dataset could have NMAR columns because the missingness is more likely to be explained by some column in your dataset. I give a handful of columns that I think could thoroughly explain most missingness so I believe the likelihood that a column in the original dataset is NMAR is low. 

## Missingness Dependency

So we mostly believe that league and other likewise data may explain the missingness of much of our dataset. Let's test out that a bit. 

Our csat10 column contains a handful of missing values, so does that information depend on the league the match took place in? To test that we run a permutation using the TVD between leagues for the proportion of the data that is missing and not missing. Since we have decided the data doesn't contain NMAR columns (and the missingness is not by design since we cannot 100% predict the missingness), this test will help decide MAR vs MCAR.
<iframe src="assets/missingness1.html" width=800 height=600 frameBorder=0></iframe> 

From the test, we get an observed TVD of 0.99. As shown above, our simulated TVDs are far below these statistics, and we obtain a p-value of 0. At a 5% threshold, we reject the null hypothesis that the data is MCAR.

We will do the same analysis for csat15 but instead, we will investigate if csat15
s missingness depends on if the team was on the blue side. 

<iframe src="assets/missingness2.html" width=800 height=600 frameBorder=0></iframe> 

Here the observed TVD is 0.0 and we obtain a p-value of one! This should make it seem the dependency of the missingness of csat10 does not depend on if the team was playing on the blue side or not.


# Hypothesiss Testing

Based on our previous EDA, it seems like taking baron first tended to correlate with teams who won more. Let's see how that matches up. 

Null hypothesis: The distributions of Winning and Losing teams taking baron were taken from the same distribution.

Alternative Hypothesis: The distributions of winning and losing teams taking baron are different distributions. 

We will run a permutation test on these categorical variables so we will use TVD as our test stat. 

<iframe src="assets/hypotest.html" width=800 height=600 frameBorder=0></iframe> 

Clear as day, we get a very large observed stat of 0.57, far outside the current distribution and likewise we obtain a p-value of 0.0. At a 5% threshold, we reject the null hypothesis! Thus it may appear that winning and losing teams take the first baron in different distributions but we cannot say for 100% certain since these tests never prove anything.
