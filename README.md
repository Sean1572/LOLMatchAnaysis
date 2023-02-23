# League of Legend Victory Anaysis

## Introduction

League of Legends is a free-to-play mutliplayer game where teams of 5 players each battle each other in the place called summoners rift. Summoners rift is where I have spent too much time as a kid. Playing the game, its natural to try to find the best way to win as the game is increditbly techical, requires great degrees of teamwork, and roughly an hour free to play a single match. It can be a long game to play and one would want to make sure thier time is worth it. So this is raises the big question we will attempt to explore today

### What game objectives in league of legends are more likely to be taken by winning teams?

To work towards answering this question, we will use a dataset provided by https://oracleselixir.com/. This dataset contains the match history of as many pro leages as possible during the 2022 pro-play season. The raw dataset contains roughly 149232 rows and 123 columns. Luckily for us and my laptop's 12 gb of RAM, we will only care for a few columns as shown below:


| Column | Type | Description|
| ---------------- | ---------| --------------------- |
| 'gameid'| Str | Unique id for each proplay match|
| 'teamid'| Str | Unique id for each team in dataset. This will be helpful for seeing overall team performance |
| 'Playername' | Str | Name of each player. If name is not known, its unknown_player but there are still nan values. This will come into play during data cleaning. |
| 'url' | Object | Depending on the source the match data came from, the dataset may contain a link to the site where the data was pulled from. We will discuss more about this when discussing potential issues with missingness in our dataset.
| League | Str | Not to be confused with league of legends, the game title, league refers to the gaming league the pro match was played under. Again, this will help use discuss potential missingingness in our dataset |
| firstblood | Bool | Which team got first blood in a match. A player getting frist blood means that player was the frist in a match to kill another player of the opposing team. Perhaps frist bloods could create early leads? | 
| firstdragon | Bool | Which team got the first dragon in a match. Killing a dragon grants the team extra powers which could give early advantages to teams. Wonder if it pays off? |
| firstherald | Bool | Which team got the first Heralds, bosses that when a team kills them, joins said team and takes down defenses of the opposing team. Perhaps winning teams like using them? |
| firstbaron | Bool | Which team got the first Baron, a massive boss that provides a huge powerup to a team that gets it late in the match. Will definetly be worth seeing what kind of advantage a baron could give |
| firsttower | Bool | Which team gets first tower. Towers are a team's defenses, breaking them down gives an opposing team easier access to a team's nexus and victory. How often do firsttowers come into play? |
| firstmidtower | Bool | Which team gets the first midlane tower. The midlane is special in paritclar becuase it is the shortest path to the other team's nexus. Could brining it down create an advantage? |
| firsttothreetowers | Bool | which team gets 3 towers frist. See firsttower and fristmidtower above for more details about towers |

# TODO SEAN ANY OTHER COLUMSN YOU WANT TO ADD

# Data Cleaning and EDA

## Data Cleaning
Before getting into any anaysis, frist we need to discuss data cleaning. The dataset itself aggrates data from serval diffrent sources ("Match History pages, lolesports.com, lpl.QQ.com, Leaguepedia, the Riot Games solo queue APIs, and more" - https://oracleselixir.com/faq). So the downloaded dataset has a few quriks to address. 

First and foremost, the dataset contains data on both overall team performance and indivual player performance. Today we care about overall team perforamnce. Since both team player and indivual player perforamnce are in the same csv file, we need to find a way to spilt that up and get the team measurements. For that we can look at the playername/playerid column of the dataset. If we group playername by gameid, count the number of players in each group then compute how often each value appears in that count, we obsreve that 10 appears 12436 times. Since we know 10 players play in any given game, then this implies there are 12436 games in the database. Now if we want to find data on each team in the dataset, and we know that there are two teams in each match, then we should expect to find some data point that appear 12436 * 2 times. The guidencd here is matches don't logically have playernames! Sure enough, counting the number of null items in the playername column add up to 12436 * 2. So our playerdata is playerdata if playername is non null and our team data is when playername is null. 

Once that more complicated data cleaning is dealt with, we can do more standard data cleaning! Frist note is that many columns needed for the anaysis are integers when they can be booleans. The result of the match being 1 for a win and 0 for a loss? Why not switch that for True if a team wins and false if they lose. This likewise was done for many of our integer based columns.

Now for null values, I have mostly decided to leave null values as is. Keep in mind, there where mutliple leagues and mutliples sources for this dataset. Some of those sources have detailed information on the matches, others may not have access to have level of data depending on where the match was, who hosted the match, etc. Changing null values in our gameplay columns columns could therefore bias our anaysis. So for now, we leave null value as is. This could bias our anaysis to teams who performance in leagues with high levels of data granularity whose own data analysis teams may benefit from the larger amount of game data and perhaps change stragerty or game outcomes when facing other teams in thier league.

## Univariate Analysis

Okay lets do some EDA on our dataset.

<iframe src="assets/csat15.html" width=800 height=600 frameBorder=0></iframe> 

## Bivaritae Analysis

<iframe src="assets/firstbaron.html" width=800 height=600 frameBorder=0></iframe> 

## Interesting Aggregates

# Assessment of Missingness

## NMAR Missingness

I am inclined to think there is not a NMAR column in our dataset primarly because we have a complete column describing the leagues each match was played in. Suppose we believe we didn't have a column to describe the missingness of our columns, therby making a few of the columns NMAR. These columns's missingness from the data generating process arises when the data associated with that match is not present in any of the data collection sites the dataset was scraped. The big question then becomes why those data points are not present in the data collection sites which bring us to the diffrent leagues. Each league is going to have diffrent policies on how much data about the match is circulated or each league has diffrent data collection agencies obsreving the match and collecting data on it. So if we wanted to determine the missinginess of a certain column, it makes the most since to know what league the match took place in so we can determine what data collection policies are in place. We do have information about the leagues so our missingness can depend on the leagues themselves.

## Missingness Dependency

<iframe src="assets/missingness1.html" width=800 height=600 frameBorder=0></iframe> 

#### TODO GET THE OTHER GRAPH IN


# Hypothesiss Testing

<iframe src="assets/hypotest.html" width=800 height=600 frameBorder=0></iframe> 
