NHL API Interface
================

  - [Required Packages](#required-packages)
  - [Write functions to contact the NHL Records
    API](#write-functions-to-contact-the-nhl-records-api)

Link to github repository

## Required Packages

``` r
library(tidyverse)
library(httr)
library(jsonlite)
library(knitr)
```

## Write functions to contact the NHL Records API

Note – to help with ease of tracking variable scope – function arguments
have the letter ‘a’ for a prefix, variables local to the function have
an ‘f’ for a prefix. Global variables have a ‘g’ prefix.

``` r
#Returns first partial matching entry, unsorted, from input conditions
pullIDDecoder<- function(aName = "", aTeamID= "", aFranchiseID = "") 
{
  #Run a pull against the franchise table to get the full decoding
  fBase<- "https://records.nhl.com/site/api/"
  fEndpoint<- "franchise"
  fCallURL<- paste0(fBase,fEndpoint)
  

  fRawJSON<- GET(fCallURL)
  fJSON_text<- content(fRawJSON, "text")
  fParsedText <- fromJSON(fJSON_text, flatten = TRUE)
  fDF <- as.data.frame(fParsedText)
  
  fTeamList<- as_tibble(fDF)%>%
    filter(toupper(data.teamCommonName) == toupper(aName) | data.id == aFranchiseID | data.mostRecentTeamId == aTeamID)
  
  fDecoder <- list("TeamID" = toString(fTeamList$data.mostRecentTeamId[1]), "FranchiseID" = toString(fTeamList$data.id[1]))
  fDecoder <- lapply(fDecoder, FUN=sub, pattern = "NA", replacement = "")
  
  return(fDecoder)
}
```

``` r
#input options for franchise common name, franchise id number, and teamid 
#function returns id, first seasonid, and last seasonid and name of every team in the history of the NHL
getFranchiseInfo<- function(aName = "", aTeamID= "", aFranchiseID = ""){
#Define API url and pull results
fBase<- "https://records.nhl.com/site/api/"
fEndpoint<- "franchise"

#Process our inputs.  If any is non-empty, we need to add a query modifier.
#Choosing to query for FranchiseID, but could easily pull on mostRecentTeamID
fCayenneExp = ""
if(aName!="" | aTeamID != "" | aFranchiseID !="") fCayenneExp <- pullIDDecoder(aName, aTeamID, aFranchiseID)$FranchiseID
#Append the appropriate prefix for this base & endpoint
if(fCayenneExp != "") fCayenneExp <-paste0("?cayenneExp=id=",fCayenneExp)

#Pull API Data
fCallURL<- paste0(fBase,fEndpoint,fCayenneExp)

#Process results into a dataframe and return
fRawJSON<- GET(fCallURL)
fJSON_text<- content(fRawJSON, "text")
fParsedText <- fromJSON(fJSON_text, flatten = TRUE)
fDF <- as.data.frame(fParsedText)

return(fDF)
}
```

``` r
#input options for franchise common name and franchise team id number, number runs slightly faster
#function Returns total stats for every franchise (ex roadTies, roadWins, etc)
getFranchiseTotals<- function(aName = "", aTeamID= "", aFranchiseID = ""){
#Define API url and pull results
fBase<- "https://records.nhl.com/site/api/"
fEndpoint<- "franchise-team-totals"

#Process our inputs.  If any is non-empty, we need to add a query modifier.
#Need to pull for FranchiseID
fCayenneExp = ""
if(aName!="" | aTeamID != "" | aFranchiseID !="") fCayenneExp <- pullIDDecoder(aName, aTeamID, aFranchiseID)$FranchiseID
#Append the appropriate prefix for this base & endpoint
if(fCayenneExp != "") fCayenneExp <-paste0("?cayenneExp=franchiseId=",fCayenneExp)

fCallURL<- paste0(fBase,fEndpoint,fCayenneExp)

#Process results into a dataframe and return
fRawJSON<- GET(fCallURL)
fJSON_text<- content(fRawJSON, "text")
fParsedText <- fromJSON(fJSON_text, flatten = TRUE)
fDF <- as.data.frame(fParsedText)

return(fDF)
}
```

``` r
getFranchiseSeasonRecords<- function(aName = "", aTeamID= "", aFranchiseID = ""){
#Define API url and pull results
fBase<- "https://records.nhl.com/site/api/"
fEndpoint<- "franchise-season-records"

#Process our inputs.  If any is non-empty, we need to add a query modifier.
#Need to pull for FranchiseID
fCayenneExp = ""
if(aName!="" | aTeamID != "" | aFranchiseID !="") fCayenneExp <- pullIDDecoder(aName, aTeamID, aFranchiseID)$FranchiseID
#Append the appropriate prefix for this base & endpoint
if(fCayenneExp != "") fCayenneExp <-paste0("?cayenneExp=franchiseId=",fCayenneExp)

fCallURL<- paste0(fBase,fEndpoint,fCayenneExp)

#Process results into a dataframe and return
fRawJSON<- GET(fCallURL)
fJSON_text<- content(fRawJSON, "text")
fParsedText <- fromJSON(fJSON_text, flatten = TRUE)
fDF <- as.data.frame(fParsedText)

return(fDF)
}
```

``` r
#Goalie records for the specified franchise 
#/franchise-goalie-records?cayenneExp=franchiseId=ID
getFranchiseGoalieRecords<- function(aName = "", aTeamID= "", aFranchiseID = ""){
#Define API url and pull results
fBase<- "https://records.nhl.com/site/api/"
fEndpoint<- "franchise-goalie-records"

#Process our inputs.  If any is non-empty, we need to add a query modifier.
#Need to pull for FranchiseID
fCayenneExp = ""
if(aName!="" | aTeamID != "" | aFranchiseID !="") fCayenneExp <- pullIDDecoder(aName, aTeamID, aFranchiseID)$FranchiseID
#Append the appropriate prefix for this base & endpoint
if(fCayenneExp != "") fCayenneExp <-paste0("?cayenneExp=franchiseId=",fCayenneExp)

fCallURL<- paste0(fBase,fEndpoint,fCayenneExp)

#Process results into a dataframe and return
fRawJSON<- GET(fCallURL)
fJSON_text<- content(fRawJSON, "text")
fParsedText <- fromJSON(fJSON_text, flatten = TRUE)
fDF <- as.data.frame(fParsedText)

return(fDF)
}
```

``` r
#/franchise-skater-records?cayenneExp=franchiseId=ID Skater records, same interaction as goalie endpoint
getFranchiseSkaterRecords<- function(aName = "", aTeamID= "", aFranchiseID = ""){
#Define API url and pull results
fBase<- "https://records.nhl.com/site/api/"
fEndpoint<- "franchise-skater-records"

#Process our inputs.  If any is non-empty, we need to add a query modifier.
#Need to pull for FranchiseID
fCayenneExp = ""
if(aName!="" | aTeamID != "" | aFranchiseID !="") fCayenneExp <- pullIDDecoder(aName, aTeamID, aFranchiseID)$FranchiseID
#Append the appropriate prefix for this base & endpoint
if(fCayenneExp != "") fCayenneExp <-paste0("?cayenneExp=franchiseId=",fCayenneExp)

fCallURL<- paste0(fBase,fEndpoint,fCayenneExp)

#Process results into a dataframe and return
fRawJSON<- GET(fCallURL)
fJSON_text<- content(fRawJSON, "text")
fParsedText <- fromJSON(fJSON_text, flatten = TRUE)
fDF <- as.data.frame(fParsedText)

return(fDF)
}
```

``` r
#Available for validation
#gdfFranchiseAll<-getFranchiseInfo()
#gdfFranchiseName<-getFranchiseInfo(aName="Devils")
#gdfFranchiseNumber<-getFranchiseInfo(aFranchiseID = 2)
#gdfFranchiseTeam<-getFranchiseInfo(aTeamID = 2)

#gdfFranTotal<-getFranchiseTotals()
#gdfFranTotalName<-getFranchiseTotals(aName="Devils")
#gdfFranTotalNumber<-getFranchiseTotals(aFranchiseID = 2)
#gdfFranTotalTeam<-getFranchiseTotals(aTeamID = 2)

#gdfFranSeas<-getFranchiseSeasonRecords()
#gdfFranSeasName<-getFranchiseSeasonRecords(aName="Devils")
#gdfFranSeasNumber<-getFranchiseSeasonRecords(aFranchiseID = 2)
#gdfFranSeasTeam<-getFranchiseSeasonRecords(aTeamID = 2)

#gdfFranGoal<-getFranchiseGoalieRecords()
#gdfFranGoalName<-getFranchiseGoalieRecords(aName="Devils")
#gdfFranGoalNumber<-getFranchiseGoalieRecords(aFranchiseID = 2)
#gdfFranGoalTeam<-getFranchiseGoalieRecords(aTeamID = 2)

#gdfFranSkat<-getFranchiseSkaterRecords()
#gdfFranSkatName<-getFranchiseSkaterRecords(aName="Devils")
#gdfFranSkatNumber<-getFranchiseSkaterRecords(aFranchiseID = 2)
#gdfFranSkatTeam<-getFranchiseSkaterRecords(aTeamID = 2)
```

``` r
#Need a different filter function for the different API set.
#This version needs to resolve 
```

``` r
ProcessExpandArgs<- function(aExpand, aTeams)
{
  fExpand<- toupper(aExpand)
  if(fExpand == "") return("")
  if(fExpand == "ROSTER") return("?expand=team.roster")
  if(fExpand == "EZROSTER") return("?expand=person.names")
  if(fExpand == "NEXTSCHED") return("?expand=team.schedule.next")
  if(fExpand == "PREVSCHED") return("?expand=team.schedule.previous")
  if(fExpand == "STATS") return("?expand=team.stats")
  if(fExpand == "TEAMLIST") return(paste0("?teamId=", aTeams))
}


#You should have a function to return parsed data for the following endpoint from the stats API(the user should be able to specify values for any of the eight modifiers listed):∗https://statsapi.web.nhl.com/api/v1/teams

#aExpand can take the following values, pulling different summary expansions:
  #"Roster", "EZRoster", "NextSched","PrevSched","Stats", "TeamList",  ""
  #Note that the TeamList value needs valid teamIDs, and that the common inputs of aName and aNumber supersede this command

#aSeason is a string formatted Y111Y222 restricting the pull to the season of interest.
#By default, an empty string will pull the current season where applicable
#Example:  20142015 to refer to the 2014-2015 hockey season.

#aTeams is a comma separated string of teamIDs and requires the 'TeamList' expansion instruction 
#Example: "4,5,29"
#Note that the common inputs of aName, aTeamID, and aFranchiseID supersede this field
getTeamStats<- function(aExpand="", aSeason = "", aTeams = "", 
                        aName = "", aTeamID= "", aFranchiseID = "")
{
  #Process our inputs.  If any is non-empty, we need to add a query modifier.
  #Check for a common input
  fTeamStr = ""
  if(aName!="" | aTeamID != "" | aFranchiseID !="")
    {fTeamStr <- pullIDDecoder(aName, aTeamID, aFranchiseID)$TeamID}
  #Append the appropriate prefix for this base & endpoint
  if(fTeamStr != "") fTeamStr <-paste0("/",fTeamStr)
  
    #Process aExpand and aTeams where relevant
    fExpand<- ProcessExpandArgs(aExpand, aTeams)
      
    #process aSeason
    fSeason=if(aSeason != ""){
      #If we already have a modifier, we need to tack this on with an &
      if(fExpand!="") paste0("&season=", aSeason)
      #else we need to use this as a direct modifier
      else paste0("?season=", aSeason)
    }

  #Define API url and pull results
  fBase<- "https://statsapi.web.nhl.com/api/v1/"
  fEndpoint<- "teams"

  fCallURL<- paste0(fBase,fEndpoint,fTeamStr,fExpand,fSeason)
  print(fCallURL)

  #Process results into a dataframe and return
  fRawJSON<- GET(fCallURL)
  fJSON_text<- content(fRawJSON, "text")
  fParsedText <- fromJSON(fJSON_text, flatten = TRUE)
  fDF <- as.data.frame(fParsedText)

  return(fDF)
}

#Available for validation
#gdfTeamStatsa<- getTeamStats(aExpand = "Roster")
#gdfTeamStatsb<- getTeamStats(aExpand = "EZRoster")
#gdfTeamStatsc<- getTeamStats(aExpand = "NextSched")
#gdfTeamStatsd<- getTeamStats(aExpand = "PrevSChed")
#gdfTeamStatse<- getTeamStats(aExpand = "Stats")
#gdfTeamStatsf<- getTeamStats(aExpand = "Teamlist", aTeams = "4,5,7")
#gdfTeamStatsg<- getTeamStats(aSeason = "20142015")
#gdfTeamStatsh<- getTeamStats(aExpand = "Stats", aSeason = "20142015")
#gdfTeamStatsi<- getTeamStats(aName = "Devils")
#gdfTeamStatsj<- getTeamStats(aFranchiseID = 2)
#gdfTeamStatsk<- getTeamStats(aTeamID = 2)
#gdfTeamStatsl<- getTeamStats(aTeamID = "2", aTeams = "4,5,7")

#gdfTeamStatsm<-getTeamStats(aName="Devils")
#gdfTeamStatsn<-getTeamStats(aFranchiseID = 2)
#gdfTeamStatso<-getTeamStats(aTeamID = 2)
#gdfTeamStatsp<-getTeamStats()
```

``` r
pullNHLAPI<- function(aEndpoint = "", ...){
  #If we don't have any inputs, print out instructions
  #Also pull the default franchise decoder
  if(aEndpoint == ""){
    print("Please rerun with a specific endpoint chosen. Options are: ")
    print("FranchiseInfo")
    print("FranchiseTotals")
    print("FranchiseSeasonRecords")
    print("FranchiseGoalieRecords")
    print("FranchiseSkaterRecords")
    print("Team")
    print("")
    print("Common arguments to the queries are:")
    print("aName - common name of the team from FranchiseInfo pull")
    print("aTeamID - MostRecentTeamID from the FranchiseInfo pull")
    print("aFranchiseID - id field from the FranchiseInfo pull.")
    print("Note 1- aFranchiseID is constant for franchises relocating or changing names")
    print("Note 2- The \"Team\" Query has additional potential modifiers, please see vignette for more details")
    print("Vignette URL")
    return(getFranchiseInfo(...))}
  
    if(aEndpoint == "FranchiseInfo"){return(getFranchiseInfo(...))}
    if(aEndpoint == "FranchiseTotals"){return(getFranchiseTotals(...))}
    if(aEndpoint == "FranchiseSeasonRecords"){return(getFranchiseSeasonRecords(...))}
    if(aEndpoint == "FranchiseGoalieRecords"){return(getFranchiseGoalieRecords(...))}
    if(aEndpoint == "FranchiseSkaterRecords"){return(getFranchiseSkaterRecords(...))}
    if(aEndpoint == "Team"){return(getTeamStats(...))}
}

#Available for validation
#gdfWrap<- pullNHLAPI()
#gdfWrapa<- pullNHLAPI("FranchiseInfo")
#gdfWrapb<- pullNHLAPI("FranchiseTotals")
#gdfWrapc<- pullNHLAPI("FranchiseSeasonRecords")
#gdfWrapd<- pullNHLAPI("FranchiseGoalieRecords")
#gdfWrape<- pullNHLAPI("FranchiseSkaterRecords")
#gdfWrapf<- pullNHLAPI("Team",aName = "Devils")
#gdfWrapg<- pullNHLAPI("Team",aTeamID = 2)
#gdfWraph<- pullNHLAPI("Team",aFranchiseID = "2")
#gdfWrapi<- pullNHLAPI("Team")
#gdfWrapf<- pullNHLAPI("Team")
```

\#\#Example Analysis  
Haven’t followed Hockey too much. From what I understand, it’s mostly
about scoring goals and checking people just hard enough that they don’t
put you in the penalty box. Wayne Gretzky was a household name growing
up. Let’s do some exploration for context about how good of a player he
was.

Data is available in the FranchiseSkaterRecords endpoint:

``` r
#Pull Data from API
gTibGretz<- as_tibble(pullNHLAPI("FranchiseSkaterRecords"))
```

    ## No encoding supplied: defaulting to UTF-8.

``` r
#Flag the records that are associated with Wayne Gretzky
gTibGretz <- gTibGretz  %>% mutate(Gretzky = ifelse(data.lastName=="Gretzky" & data.firstName == "Wayne",
         "Wayne Gretzky", "Rest of League"))
```

First things first, there are probably a lot of alternates, rookies, and
other folks that didn’t get a lot of time on the ice in this pull. Let’s
take a look at the distribution of games played per season and filter
them out.

``` r
gTibGretz<- gTibGretz %>% mutate (GamesPerSeason = data.gamesPlayed / data.seasons)
g <- ggplot(data = gTibGretz, mapping = aes(x = GamesPerSeason))
g+ geom_histogram(binwidth = 1)+ggtitle("Games Per Season for Player Franchise Records")
```

![](README_files/figure-gfm/Low%20Playtime-1.png)<!-- -->

``` r
#It's a fairly uniform density, but it looks like a second mode starts around 30 games.
#Let's filter out entries with <30 games/ season
gTibGretz <- gTibGretz %>% filter(GamesPerSeason >= 30)
```

Ok, now let’s figure out how scoring goals correlates to penalty time.

``` r
#Scatterplot to evaluate bivariate relationship
g<- ggplot(data = gTibGretz, mapping = aes (x= data.mostPenaltyMinutesOneSeason, y = data.mostGoalsOneSeason))
g+geom_point(mapping = (aes(colour = data.positionCode, shape = Gretzky, size = Gretzky))) + 
  ggtitle("Personal Bests at a Franchise, Goals vs Penalty Time by Player, All Time")
```

    ## Warning: Using size for a discrete variable is not advised.

![](README_files/figure-gfm/Penalty%20vs%20Scoring-1.png)<!-- --> We see
that Wayne had a clearly phenomenal tenure at one of his franchises,
apparently still being the all time scoring leader for goals in a
regular season by a significant margin across all franchises. He was
also apparently very good at not getting caught checking people, as his
maximum penalty time at that franchise was less than any other center
who scored more than 63 goals in one season.

It also looks like there are four different clusters of exceptional
players. We have the Gretzky Group, who are covert and high-scoring.
These players score over 63 times in a season and have stayed under 100
minutes of penalties in a season. We also have the stadium fillers, this
group is aggressive but score goals in the process. They hang out in the
penalty box 200-300 minutes in a season while scoring more than 38 goals
in their best seasons. Franchise owners probably love them, coaches
probably hate them. Second to last we have the defense and left wingers.
These guys just like hitting people and presumably remember to clear the
puck from time to time. Finally, we have some guy who managed to rack up
6 hours of penalty time in a single season – one whole hour more than
any other player has ever been tolerated by their coach. Dave Schultz,
from – you guessed it – Philadelphia.

Here’s the top 6 players for penalty times in a season:

``` r
#Players with the most penalty minutes in a single season.
kable(head(gTibGretz%>% select(data.firstName, data.lastName, data.franchiseName, data.mostPenaltyMinutesOneSeason,            data.mostPenaltyMinutesSeasonIds)%>% arrange(desc((data.mostPenaltyMinutesOneSeason)))))
```

| data.firstName | data.lastName | data.franchiseName  | data.mostPenaltyMinutesOneSeason | data.mostPenaltyMinutesSeasonIds |
| :------------- | :------------ | :------------------ | -------------------------------: | :------------------------------- |
| Dave           | Schultz       | Philadelphia Flyers |                              472 | 19741975                         |
| Paul           | Baxter        | Pittsburgh Penguins |                              409 | 19811982                         |
| Mike           | Peluso        | Chicago Blackhawks  |                              408 | 19911992                         |
| Marty          | McSorley      | Los Angeles Kings   |                              399 | 19921993                         |
| Bob            | Probert       | Detroit Red Wings   |                              398 | 19871988                         |
| Basil          | McRae         | Dallas Stars        |                              382 | 19871988                         |

Alright, back to Wayne Gretzky. We just saw that he was a center. How
many skaters from different positions are we comparing against? We
wouldn’t want to compare that position against a defenseman for scoring.
Let’s generate average penalty time per game and and average goals per
game to see how the different positions compare.

``` r
#Mutate in Avg Penalty Time and Avg Goals per Game at a franchise
gTibGretz <-gTibGretz %>% mutate( penaltyMinPerGame = data.penaltyMinutes / data.gamesPlayed,
                                   goalsPerGame = data.goals/data.gamesPlayed)

gTibPositionSummary<- gTibGretz %>% group_by(data.positionCode, Gretzky) %>% 
  summarise(nPlayerFranchiseObs = n(), avgGoalsPerGame=mean(goalsPerGame),
            stdDevGoals = sd(goalsPerGame), avgPenaltyBoxPerGame = mean(penaltyMinPerGame),
            stdDevPenaltyBoxPerGame = sd(penaltyMinPerGame))
```

    ## `summarise()` regrouping output by 'data.positionCode' (override with `.groups` argument)

``` r
#Barchart vs Position
g<- ggplot(data = gTibPositionSummary, mapping= aes(x = data.positionCode,
                                        y = nPlayerFranchiseObs, fill = data.positionCode))
g + geom_bar( stat = 'identity' ) + ggtitle('Count of players vs position') +
  ylab('Number of Players This Position') + xlab('Position')
```

![](README_files/figure-gfm/Numerical%20Summaries-1.png)<!-- -->

``` r
#Performance Summary for each position
kable(gTibPositionSummary, digits = 3)
```

| data.positionCode | Gretzky        | nPlayerFranchiseObs | avgGoalsPerGame | stdDevGoals | avgPenaltyBoxPerGame | stdDevPenaltyBoxPerGame |
| :---------------- | :------------- | ------------------: | --------------: | ----------: | -------------------: | ----------------------: |
| C                 | Rest of League |                2197 |           0.191 |       0.106 |                0.572 |                   0.454 |
| C                 | Wayne Gretzky  |                   3 |           0.513 |       0.301 |                0.367 |                   0.086 |
| D                 | Rest of League |                2924 |           0.066 |       0.051 |                0.981 |                   0.675 |
| L                 | Rest of League |                1824 |           0.189 |       0.109 |                0.888 |                   0.840 |
| R                 | Rest of League |                1774 |           0.203 |       0.120 |                0.814 |                   0.734 |

From that quick summary, we see that Centers, Left Wingers, and Right
Wingers are all roughly consistent for number of points scored.
Defensemen understandably score far less. Just from the numerical
summary, we do see that Gretzky averages 3 standard deviations over the
league average for a center, quite impressive. The distribution of
penaltybox times is far noisier with a coefficient of variance close to
1. Gretzky is \~36% below average compared to the other Centers.

Let’s continue to look at where Gretzky falls relative to other
offensive players.

``` r
#Trim the defensive players from the set
gTibGretz <- gTibGretz %>% filter(data.positionCode%in% c('C', 'R', 'L'))

#Setup common variables for plotting
g<- ggplot(data = gTibGretz, mapping = aes(x = Gretzky, color = Gretzky))

#Individual Box Plots for several metrics
g + geom_boxplot(mapping = aes(y = goalsPerGame)) + ggtitle('Goals per Game') +
  geom_jitter(mapping = aes(y = goalsPerGame) )
```

![](README_files/figure-gfm/boxplots-1.png)<!-- -->

``` r
g + geom_boxplot(mapping = aes(y = data.mostGoalsOneGame)) + ggtitle('Most Goals One Game') + 
  ylab('Goals in a Single Game') + geom_jitter(mapping = aes(y = data.mostGoalsOneGame))
```

![](README_files/figure-gfm/boxplots-2.png)<!-- -->

``` r
g + geom_boxplot(mapping = aes(y = data.mostAssistsOneGame)) + ggtitle('Most Assists One Game') + 
  ylab('Assists in a Single Game') + geom_jitter(mapping = aes(y =data.mostAssistsOneGame))
```

![](README_files/figure-gfm/boxplots-3.png)<!-- -->

``` r
g + geom_boxplot(mapping = aes(y = data.mostPointsOneGame)) + 
  ggtitle('Most Points One Game (Goals + Assists)') + ylab('Points in a Single Game') +
  geom_jitter(mapping = aes(y = data.mostPointsOneGame))
```

![](README_files/figure-gfm/boxplots-4.png)<!-- -->

``` r
g + geom_boxplot(mapping = aes(y = penaltyMinPerGame)) + ggtitle('Average Penalty Time per Game (minutes)') + 
  ylab('Penalty Time per Game') + geom_jitter(mapping = aes(y = penaltyMinPerGame))
```

![](README_files/figure-gfm/boxplots-5.png)<!-- --> Clearly an exemplary
player with numbers that are matched by few across his tenures at
several franchises. Let’s see how his franchises performed

``` r
#Figure out which teams Gretzky Played on, and filter data table down to those franchises
gTibGretzFranchList<- gTibGretz %>% group_by(Gretzky, data.franchiseId, data.franchiseName) %>%
  summarise(offensiveSkaterCount = n(), player.gamesPlayed = max(data.gamesPlayed))
```

    ## `summarise()` regrouping output by 'Gretzky', 'data.franchiseId' (override with `.groups` argument)

``` r
#Print the teams
kable(gTibGretzFranchList%>%select(-offensiveSkaterCount)%>%filter(Gretzky =="Wayne Gretzky"))
```

| Gretzky       | data.franchiseId | data.franchiseName | player.gamesPlayed |
| :------------ | ---------------: | :----------------- | -----------------: |
| Wayne Gretzky |               10 | New York Rangers   |                234 |
| Wayne Gretzky |               14 | Los Angeles Kings  |                539 |
| Wayne Gretzky |               25 | Edmonton Oilers    |                696 |

``` r
#Join our Gretzky decoder to the franchise totals
gTibGretzkyFranch<-gTibGretzFranchList%>%
  inner_join(pullNHLAPI("FranchiseSeasonRecords"), by = "data.franchiseId")
```

    ## No encoding supplied: defaulting to UTF-8.

``` r
gTibFranchSumm <- gTibGretzkyFranch %>% group_by(Gretzky) %>%
  summarise(franchiseCount = n(), averageMostGoals = mean(data.mostGoals), 
            stdDevMostGoals = sd(data.mostGoals), averageMostWins = mean(data.mostWins),
            stdDevMostWins = sd(data.mostWins), averagePenaltyTime = mean(data.mostPenaltyMinutes),
            stdDevPenaltyTime = sd(data.mostPenaltyMinutes))
```

    ## `summarise()` ungrouping output (override with `.groups` argument)

``` r
kable(gTibFranchSumm, digits = 1)
```

| Gretzky        | franchiseCount | averageMostGoals | stdDevMostGoals | averageMostWins | stdDevMostWins | averagePenaltyTime | stdDevPenaltyTime |
| :------------- | -------------: | ---------------: | --------------: | --------------: | -------------: | -----------------: | ----------------: |
| Rest of League |             37 |            303.7 |            86.2 |            48.4 |           11.8 |             1852.9 |             698.9 |
| Wayne Gretzky  |              3 |            381.0 |            62.6 |            52.7 |            4.5 |             2147.0 |             115.2 |

``` r
g<- ggplot(data = gTibGretzkyFranch, mapping = aes(x=Gretzky))

g+geom_boxplot(mapping = aes(y = data.mostGoals)) + ggtitle("Most Franchise Goals in a Season -- All Time") + 
  geom_jitter(mapping =aes(y=data.mostGoals)) 
```

![](README_files/figure-gfm/Analysis_2-1.png)<!-- -->

``` r
g+geom_boxplot(mapping = aes(y = data.mostWins)) + ggtitle("Most Franchise Wins in a Season -- All Time") + 
  geom_jitter(mapping =aes(y=data.mostWins)) + ylab('Most Wins')
```

![](README_files/figure-gfm/Analysis_2-2.png)<!-- --> These plots are a
stretch, since we’re not actually limited to the seasons where Gretzky
was and was not playing for these franchises – we’re looking at all time
records. Still, I’d bet an all-time leading scorer was on the roster for
at least of those franchise goal records. For wins in a season, it looks
like Gretzky’s best team is in the top 7 while the others are more
typical.

For one last look at his influence, let’s see how many skaters were
named Wayne in the 2000s.

``` r
#Pull a fresh, untrimmed set of skater data.  
gTibWayne<- as_tibble(pullNHLAPI("FranchiseSkaterRecords"))
```

    ## No encoding supplied: defaulting to UTF-8.

``` r
#Add two factor columns for name and rough date ~20 years after Gretzky joined the Oilers
gTibWayne<- gTibWayne %>%  mutate(NamedWayne = ifelse(data.firstName == 'Wayne',"Wayne", "Not Wayne"), 
         After2000 = ifelse(as.numeric(substr(data.mostPointsSeasonIds,1,4)) > 2000, 
                            "Later than 2000", "Before 2000"))

table(gTibWayne$NamedWayne, gTibWayne$After2000)
```

    ##            
    ##             Before 2000 Later than 2000
    ##   Not Wayne       10038            6759
    ##   Wayne              64              10

Well, it looks like the name Wayne has not shot up in popularity in the
years since Gretzky became famous. I guess that just makes Gretzky that
much more unique.
