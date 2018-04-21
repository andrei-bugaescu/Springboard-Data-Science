# Final Report

## Problem
Predicting the outcomes of sporting events is a staple activity in sports that was more art than skill for a long time. It wasn't until the advent of Sports Analytics that the methodology behind sports prediction became quantified. With data becoming increasingly available to everyone, it's now possible for anyone to create a model to predict the outcomes of sports. 

The highly variant nature of college basketball presents an interesting case; the NCAA tournament is dubbed "March Madness" after all. [Kaggle](https://www.kaggle.com/c/mens-machine-learning-competition-2018/leaderboard) hosts yearly competitions for predicting March Madness but very few models consistently perform well. Vegas and the Oddsmakers are still the gold standard in predicting sports and March Madness is no different. After all, if they weren't, they wouldn't be profitable. Like a small-school underdog facing a blue-blood powerhouse, this model seeks to answer the question: Can we beat the odds?

### A Quick Summary of the NCAA Tournament
The NCAA Tournament is a 68 team, sudden-death tournament. An example is shown below.

![Sample NCAA tourney](https://www.printyourbrackets.com/images/ncaa-march-madness-results-2018.png)

The first four games are "play-in" games that 8 teams play in order to earn the last 4 spots of the main bracket, which is a 64 team bracket. Each round, the remaining teams play a game where the winning team advances to the next round and the losing team is eliminated. The teams who perform the best during the regular season are awarded the best (low) seeds, and get to face the weaker teams which barely make the tournament that are awarded high seeds. There are 6 rounds, chronologically referred to as "1st Round/The Round of 64", "2nd Round/The Round of 32", "3rd Round/Sweet 16/Regional Semifinals", "4th Round/Elite Eight/Regional Finals", "5th Round/Final Four/Tournament Semifinals", "6th Round/National Championship/Tournament Finals". 

### Explaining Betting Lines
Resources: [1](https://www.oddsshark.com/sports-betting/point-spread-betting) 
[2](https://www.pinnacle.com/en/betting-articles/Basketball/Basketball-Bet-Types-Explained/HST2G2NVF267NTS3) (See Handicap Betting)
[3](https://www.quora.com/What-does-covering-the-spread-mean-in-sports-betting) The convention used in the Spurs/Lakers example is flipped in this case. Here, favored teams have positive point spreads.

Suppose two teams play each other, Team A and Team B. Team A is perceived to be stronger than Team B. Oddsmakers will attempt to gauge how much stronger Team A is than Team B by setting how many points Team A is favored to win over Team B. This estimation of how many points Team A is favored by is the "Point spread", or betting line.

Now suppose that the oddsmakers set the betting line to be +5.5 for Team A, which is equivalent to -5.5 for Team B. This means that the oddsmakers value that Team A is a 5 point favorite over Team B. If one bets that Team A will win by 5 points or less, or for Team B to win, he is betting for Team A, the favorite, to "lose against the spread (ATS)". This is also referred to Team B beating the spread. If another bets that Team A will win by more than 5 points, he is betting for the favorite to "win against the spread". This time, it's Team A beating the spread. 

#### Convention
For here on out, all actions against the spread are by the favorite team. So, the plots labeled are with the favorite winning or losing against the spread. 

### Objective: Create a Model That Suggests a Profitable Betting Strategy
The goal of this project is to create a model that can:
1. Accurately predict the outcome of games.
2. Using the predicted outcome, identify bets to make according the Vegas betting lines and develop a profitable betting strategy.

## Clients
### The Underdog Mid-Majors: Sports Betters
Enthusiastic sports betters would be highly interested in a model that gives them an edge over the house. A well-performing model could yield lucrative returns on betting in the long run.

### The Power Conference Staples: Sports Analytics Companies
Sports Analytics companies would be interested in this type of model as well. The companies are often contracted by actual sports teams looking to gain an edge over the competition through analytics.

### The Blue-Bloods: The Oddsmakers
If this model is found to consistently beat The Oddsmakers, then it would be in their best interest to maintain a house advantage over prospective betters by implementing aspects of a high performing model. 

## Initial Data

### Basketball Data - [Kaggle NCAA ML Competition - Men's](https://www.kaggle.com/c/mens-machine-learning-competition-2018/data) 
The basketball data used for this analysis comes from the 2018 March Madness competition hosted by Kaggle. The most important pieces of data include:
- RegularSeason(Compact, Detailed)Results.csv: Contains the results of NCAA regular season games. The compact version contains the winning and losing Team ID's, the season, and score of results since 1985. The detailed version contains all of the columns for the compact version with two differences:
  - The earliest season is 2003.
  - Each row contains boxscore stats for each game, including but not limited to: Field Goals Attempted/Made (FGA/FGM), Rebounds (REB), Blocks (BLK), Steals (STL), Three-Pointers Attempted/Made (TPA/TPM), etc. of each team.
 Advanced statistics that will be feature engineered in the future can only be calculated with the detailed version, so that version is used. The dimensions are (82041 x 8). 
- NCAATourney(Compact, Detailed)Results.csv: Contains the results of NCAA Tournament games. Similar in structure to the Regular Season .csv files, but for tournament games. The results are the key feature for this .csv file so the compact version is used. The dimensions are (1063 x 34)
- Teams.csv and TeamSpellings.csv: Each team is identified by an integer Team ID. Teams.csv contains the most common name of a school in one column, and the corresponding ID in another. For example, "Michigan State University" has a team ID of 1277. TeamSpellings.csv contains common variations of team spellings in one column, and the corresponding ID in another. For example, Michigan State University can also be spelled as "Mich. State University", "Mich-State-University", etc., so each value of those rows would still correspond to the Team ID for Michigan State University, 1277. This is an important piece in future data-wrangling tasks as explained below. 

### Betting Data - [The Prediction Tracker](http://thepredictiontracker.com/)
This dataset contains historical betting lines for many sports including NCAA basketball. There's a spreadsheet for each season and they're named with the convention 'ncaabb{last 2 numbers of season}.csv' For example, the betting odds for the 2017-2018 season is 'ncaabb17.csv'. Each spreadsheet contains the following features:
- Each row is a game. Typically there are ~4000 rows per season.
- There first 5 columns are columns about the game itself, such as the home/road team (home, road), the home/road team score (hscore, rscore) and date.
- The rest of the columns contain information about the betting lines. These have the prefix of "line-" and are followed by the platform that provides these lines. For example, the betting line for the "Fox" platform would be listed under the "linefox". These values are often rounded to the nearest 0.5, but if they're not, they're wrangled in one of the data cleaning steps.

## Data Wrangling

### Basketball Data
This dataset is largely cleaned thanks to Kaggle. It contains just about every basic feature I need. Much of the remaining work is to engineer new features by calculating various advanced metrics from this dataset. 

### Betting Data
#### Assigning Team ID's to Team Names
Each of the betting spreadsheets don't come with the Team ID's generated by the Kaggle dataset. Using `TeamSpellings.csv`, I managed to translate most (~99%) of the teams to their corresponding ID's. The ones that weren't translated were obscure schools that don't factor into the NCAA Tournament anyway, and information on those games are dropped.

#### Dropping Line Columns Populated by a Large Amount of NaNs
Columns containing lines that have >10% NaN values were dropped. This also serves as a filter for more reputable betting platforms, as the more reliable ones will be more likely to have clean data.


## [Initial Findings (Using 2017-18 Regular Season Data)](https://github.com/victoreram/Springboard-Data-Science/blob/master/NCAABBPrediction/DataExploration.ipynb)
### Betting Lines are Normally Distributed
![Residuals](https://raw.githubusercontent.com/victoreram/Springboard-Data-Science/master/NCAABBPrediction/Documents/margin_residuals.png)

Based on the residuals and the histograms below, both the betting lines and actual results are normally distributed. The following statistics were calculated for each distribution:

```
Mean betting line for home team: 4.890924741760944 

Mean margin for home team: 5.121495327102804

Standard deviation betting line for home team: 8.616574535963725

Standard deviation result for home team: 13.986181735365248
```
### Betting Lines are Closely Related To Actual Results
![Betting Odds](https://raw.githubusercontent.com/victoreram/Springboard-Data-Science/master/NCAABBPrediction/Documents/odds_distribution.png)

Plotting betting line vs. actual margin of victory shows a correlation (r=0.6) with a very low p-value (<0.01).


### Betting Against The Spread Was Profitable in the 2018 Regular Season
![Betting Against The Spread](https://raw.githubusercontent.com/victoreram/Springboard-Data-Science/master/NCAABBPrediction/Documents/bets.png)

Simply betting for the favorite to beat the spread was profitable 2017-18 regular season. However, this comes with some caveats. 

1. This is only for one regular season. Other seasons may show different results.
2. Given the high volume of games, this is not all that profitable of a strategy. Suppose you bet $100 on each game using this strategy and comes ahead by 86 games out of 4066. Suppose also that the casino is generous and give a 1:1 payout, i.e. you lose $100 if you lose the bet and the casino give you $100 in addition to the $100 you put in if you win (which is never the case!). This betting strategy amounts to a paltry ~$2.11 profit per game!

## Blind Model - Evaluating on 2017-2018 Tournaments

What this model is actually being evaluated on is its performance on predicting the NCAA tournaments and if it outperform the Vegas oddsmakers. To begin, below is a plot of how well the Vegas betting line correlates with the actual results of each tournament game.

![Betting Naive](https://raw.githubusercontent.com/victoreram/Springboard-Data-Science/master/NCAABBPrediction/Documents/mov_vs_line.png)

Then, given the betting line for the game, set a betting strategy. The first betting strategies explored are the naive models of always betting against or covering the spread is applied to the 2017 and 2018 tournaments. The results of this strategy are shown below:

![Betting Naive](https://raw.githubusercontent.com/victoreram/Springboard-Data-Science/master/NCAABBPrediction/Documents/bets_tourney.png)

This time, betting for the favorite to lose against the spread in the NCAA Tournament was profitable. This is reasonable because upsets, or games where the underdog wins, are more frequent in the NCAA tournament. Applying the betting payout above, the most profitable strategy amounts to 100*(6 games ahead/134 games) ~ $4.45 profit/game.

## Using Supervised Learning to Predict Margin of Victory

### Main Feature: [Elo Ratings](https://en.wikipedia.org/wiki/Elo_rating_system)
The main feature for this model are the Elo Ratings. Developed initially as a way to calculate skill levels for players in zero-sum games, Elo Ratings are a comprehensive metric that estimates a basketball team's strength. This model is similar to how [FiveThirtyEight](https://fivethirtyeight.com/features/how-we-calculate-nba-elo-ratings/) calculates their Elo Ratings. A team's Elo Rating is recalculated each game and takes into account:
- the margin of victory/defeat
- the strength of the teams facing each other 
- homecourt advantage
Because this rating system is zero-sum, the losing team loses Elo points equal to the Elo points the winning team gains. Thus, the average Elo Rating, which is set to 1500, is stable across all seasons. The Elo Rating system has many more nuances that are explained in the above links.

The scale for Elo Ratings are estimated as follows:

- **>2000**: Elite, championship caliber. Reserved for the very best teams, typically top 10, in college basketball. This corresponds to the 1-3 seeds. 
- **1801-2000**: Very Good/Great; a tier below elite. Teams in this category largely make up the stronger half of the tournament, the 3-8 seeds.
- **1601-1800**: Good / Above average. Teams in this category largely make up the weaker half of the tournament, the 8-13 seeds. 
- **1401-1600**: Average. Teams in the tournament that fall in this range are typically among the last ones in and are high seeds (14-16). 
- **1201-1400**. Below Average. Teams in this caregory have no business being in the tournament.
- **<1200**. Bad. These teams are typically the bottomfeeders of the league. 

#### [Tuning Elo Parameters](https://github.com/victoreram/Springboard-Data-Science/blob/master/NCAABBPrediction/EloTuning.ipynb)

Elo Ratings require two parameters to be set in order to be calculated: K and H. 

K indicates how much Elo points are exchanged in each game. An Elo Rating method with a high K value will greatly account for new data and thus be more volatile. Inversely, an Elo Rating method with a low K value won't change as much as a result of a game.

H is for Homecourt Advantage. Teams with homecourt have a 3-4 point advantage over away teams, which accounts to roughly 100-150 Elo points. Models with high H value Homecourt Advantage and thus set a higher handicap for home teams. This also has the effect of road teams winning more Elo points from the home team if it wins, and therefore accounts for the ability for a team to win away from home. Models with a low H value have the opposite effect. 

The range of K and H values observed for tuning were from 20-80 and 100-150 respectively. In order to find the best K and H, the following steps were performed:
1. Calculate each team's Elo Rating for a given K and H at the end of the regular season.
2. In each tournament game from 2003-2018, calculate the difference in Elo Ratings between the two teams facing each other, denoted as `SeasonEloDiff`.
3. Train a linear regression from 2003-2016 where X is `SeasonEloDiff` and the observable y is the actual margin of victory between the two teams.
4. Test this model on the 2017-2018 tournaments and calculate the Mean Squared Error (MSE).
5. Choose the optimal K and H as the model which minimizes MSE. 

It was found that the Elo Rating which minimized MSE were calculated with parameters K = 65 and H = 130, as shown below. H did not affect the MSE that much. Only the K value with the optimal H value is plotted below. 

![MSE vs K](https://raw.githubusercontent.com/victoreram/Springboard-Data-Science/master/NCAABBPrediction/Documents/MSE_vs_K.png)


#### Linear Regression With Optimal Elo Ratings

The baseline supervised model is the linear regression model outlined above where feature column X = `SeasonEloDiff` calculated with the optimal parameters above, and y = margin of victory. This model is trained with the results from the 2003-2016 NCAA tournaments and tested with the 2017-2018 tournaments. The results of the regression model are shown below.

![Betting EloLR](https://raw.githubusercontent.com/victoreram/Springboard-Data-Science/master/NCAABBPrediction/Documents/mov_vs_elo.png)

Elo Ratings by themselves turn out to be a solid indicator of margin of victory. Give the coefficient of ~0.03, it was found that ~33. Elo points were worth 1 point in margin of victory. This model is evaluated one step further against the average betting line.

![Betting EloLR](https://raw.githubusercontent.com/victoreram/Springboard-Data-Science/master/NCAABBPrediction/Documents/bets_tourney_lr.png)

The baseline model was able to craft a more profitable betting strategy than simply betting for the favorite to beat or lose ATS. Coming ahead by 13 games out of 134, this amounts to a profit of $100*(13/134) = $9.70/game. Not bad!

### Linear Regression With Elo Ratings And Advanced Stats

### Random Forests With Elo Ratings Adn Advanced Stats

## Evaluating Against Vegas Betting Line

## Conclusions
