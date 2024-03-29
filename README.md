NHL API Interface
================

  - [Overview](#overview)
  - [Repository and Blog](#repository-and-blog)
  - [Required Packages](#required-packages)
  - [Project Functions](#project-functions)
      - [Helper Function](#helper-function)
      - [Functions to contact the NHL Records
        API](#functions-to-contact-the-nhl-records-api)
          - [Franchise Info](#franchise-info)
          - [Franchise Totals](#franchise-totals)
          - [Franchise Season Records](#franchise-season-records)
          - [Franchise Goalie Records](#franchise-goalie-records)
          - [Franchise Skater Records](#franchise-skater-records)
          - [Sample NHL Records Function
            Calls](#sample-nhl-records-function-calls)
          - [NHL Stats Pull](#nhl-stats-pull)
          - [Sample calls for Stats API
            validation](#sample-calls-for-stats-api-validation)
      - [Wrapper Function](#wrapper-function)
          - [Wrapper Function to Combine Previous
            Endpoints](#wrapper-function-to-combine-previous-endpoints)
          - [Wrapper Validation Calls](#wrapper-validation-calls)
  - [Analysis - Wayne Gretzky
    Performance](#analysis---wayne-gretzky-performance)
      - [Initial Pull](#initial-pull)
      - [Filter For Players with Several
        Games](#filter-for-players-with-several-games)
      - [Penalty Time vs Scoring](#penalty-time-vs-scoring)
      - [Performance vs Position
        Played](#performance-vs-position-played)
      - [Offensive Player Statistics](#offensive-player-statistics)
      - [Hockey Players Named Wayne](#hockey-players-named-wayne)
  - [Recap](#recap)

# Overview

This vignette implements several functions to access the NHL API using
R, and rounds through an example analysis of Wayne Gretzky’s franchise
stats. This is a limited implementation of the API, and far more data is
available for future development and analysis.

Note when viewing this code– to help with ease of tracking variable
scope – function arguments have the letter ‘a’ for a prefix, variables
local to the function have an ‘f’ for a prefix. Global variables have a
‘g’ prefix.

# Repository and Blog

[Repository](https://github.com/fd54386/NHL-API-Interface)  
[Blogpost](https://fd54386.github.io/)

# Required Packages

To run this code, you will need to install the tidyverse, httr,
jsonlite, and knitr packages

``` r
library(tidyverse)
library(httr)
library(jsonlite)
library(knitr)
```

# Project Functions

### Helper Function

This function would be private if this were a class. It’s used to tie
the common arguments of `aName`, `aTeamId`, and `aFranchiseId` together.

``` r
#Returns first partial matching entry, unsorted, from input conditions
pullIDDecoder<- function(aName = "", aTeamId= "", aFranchiseId = "") 
{
  #Run a pull against the franchise table to get the full decoding
  fBase<- "https://records.nhl.com/site/api/"
  fEndpoint<- "franchise"
  fCallURL<- paste0(fBase,fEndpoint)
  
  #Parse results to dataframe
  fRawJSON<- GET(fCallURL)
  fJSON_text<- content(fRawJSON, "text")
  fParsedText <- fromJSON(fJSON_text, flatten = TRUE)
  fDF <- as.data.frame(fParsedText)
  
  #Filter for the called arguments
  fTeamList<- as_tibble(fDF)%>%
    filter(toupper(data.teamCommonName) == toupper(aName) | data.id == aFranchiseId |
             data.mostRecentTeamId == aTeamId)
  
  #If we didn't get any rows returned from that filter (ie -- no arguments specified)
  #Trim the NA values
  fDecoder <- list("TeamID" = toString(fTeamList$data.mostRecentTeamId[1]), "FranchiseID" = 
                     toString(fTeamList$data.id[1]))
  fDecoder <- lapply(fDecoder, FUN=sub, pattern = "NA", replacement = "")
  #Return the team and franchise Ids, need both for different endpoints
  return(fDecoder)
}
```

## Functions to contact the NHL Records API

We have written 6 functions to contact the NHL API. There is also a
wrapper function to help access all of the endpoints from a single
function if you don’t like autocomplete.

There are 3 common arguments to all of these functions. `aName` refers
to the team common name, found in the records Franchise endpoint.
`aFranchiseId` and `aTeamId` are different keys for teams.
`aFranchiseId` will be constant across name changes and city relocations
while `aTeamId` changes for each. All have comparable runtime, and will
filter for only 1 result.

### Franchise Info

This is the default call of the wrapper function. It’s a convenient
decoder for common name, franchiseID (first ID column) and TeamID
(data.mostRecentTeamID)

``` r
#function returns id, first seasonid, and last seasonid and name of every team in the history of the NHL
getFranchiseInfo<- function(aName = "", aTeamId= "", aFranchiseId = ""){

#Process our inputs.  If any is non-empty, we need to add a query modifier.
#Choosing to query for FranchiseID, but could easily pull on mostRecentTeamID
fCayenneExp = ""
if(aName!="" | aTeamId != "" | aFranchiseId !="") fCayenneExp <- pullIDDecoder(aName, aTeamId, aFranchiseId)$FranchiseID
#Append the appropriate prefix for this base & endpoint
if(fCayenneExp != "") fCayenneExp <-paste0("?cayenneExp=id=",fCayenneExp)

#Define API url and pull results
fBase<- "https://records.nhl.com/site/api/"
fEndpoint<- "franchise"

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

### Franchise Totals

This endpoint tracks the accumulated stats over the entirety of a
franchise’s existence.

``` r
#function Returns total stats for every franchise (ex roadTies, roadWins, etc)
getFranchiseTotals<- function(aName = "", aTeamId= "", aFranchiseId = ""){


#Process our inputs.  If any is non-empty, we need to add a query modifier.
#Need to pull for FranchiseID
fCayenneExp = ""
if(aName!="" | aTeamId != "" | aFranchiseId !="") fCayenneExp <- pullIDDecoder(aName, aTeamId, aFranchiseId)$FranchiseID
#Append the appropriate prefix for this base & endpoint
if(fCayenneExp != "") fCayenneExp <-paste0("?cayenneExp=franchiseId=",fCayenneExp)

#Define API url and pull results
fBase<- "https://records.nhl.com/site/api/"
fEndpoint<- "franchise-team-totals"

fCallURL<- paste0(fBase,fEndpoint,fCayenneExp)

#Process results into a dataframe and return
fRawJSON<- GET(fCallURL)
fJSON_text<- content(fRawJSON, "text")
fParsedText <- fromJSON(fJSON_text, flatten = TRUE)
fDF <- as.data.frame(fParsedText)

return(fDF)
}
```

### Franchise Season Records

This endpoint tracks the best season records for a franchise.

``` r
getFranchiseSeasonRecords<- function(aName = "", aTeamId= "", aFranchiseId = ""){
#Define API url and pull results
fBase<- "https://records.nhl.com/site/api/"
fEndpoint<- "franchise-season-records"

#Process our inputs.  If any is non-empty, we need to add a query modifier.
#Need to pull for FranchiseID
fCayenneExp = ""
if(aName!="" | aTeamId != "" | aFranchiseId !="") fCayenneExp <- pullIDDecoder(aName, aTeamId, aFranchiseId)$FranchiseID
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

### Franchise Goalie Records

This endpoint tracks goalie statistics during their tenure with a given
francies. Goalies will have multiple records in this endpoint across any
different franchises they played with.

``` r
#Goalie records for the specified franchise 
getFranchiseGoalieRecords<- function(aName = "", aTeamId= "", aFranchiseId = ""){


#Process our inputs.  If any is non-empty, we need to add a query modifier.
#Need to pull for FranchiseID
fCayenneExp = ""
if(aName!="" | aTeamId != "" | aFranchiseId !="") fCayenneExp <- pullIDDecoder(aName, aTeamId, aFranchiseId)$FranchiseID
#Append the appropriate prefix for this base & endpoint
if(fCayenneExp != "") fCayenneExp <-paste0("?cayenneExp=franchiseId=",fCayenneExp)

#Define API url and pull results
fBase<- "https://records.nhl.com/site/api/"
fEndpoint<- "franchise-goalie-records"

fCallURL<- paste0(fBase,fEndpoint,fCayenneExp)

#Process results into a dataframe and return
fRawJSON<- GET(fCallURL)
fJSON_text<- content(fRawJSON, "text")
fParsedText <- fromJSON(fJSON_text, flatten = TRUE)
fDF <- as.data.frame(fParsedText)

return(fDF)
}
```

### Franchise Skater Records

This endpoint tracks single game, season, and overall statistics for
offensive and defensive players during their tenures with a franchise.
Players will have multiple entries if they played for multiple
franchises

``` r
getFranchiseSkaterRecords<- function(aName = "", aTeamId= "", aFranchiseId = ""){
#Process our inputs.  If any is non-empty, we need to add a query modifier.
#Need to pull for FranchiseID
fCayenneExp = ""
if(aName!="" | aTeamId != "" | aFranchiseId !="") fCayenneExp <- pullIDDecoder(aName, aTeamId, aFranchiseId)$FranchiseID
#Append the appropriate prefix for this base & endpoint
if(fCayenneExp != "") fCayenneExp <-paste0("?cayenneExp=franchiseId=",fCayenneExp)

#Define API url and pull results
fBase<- "https://records.nhl.com/site/api/"
fEndpoint<- "franchise-skater-records"

fCallURL<- paste0(fBase,fEndpoint,fCayenneExp)

#Process results into a dataframe and return
fRawJSON<- GET(fCallURL)
fJSON_text<- content(fRawJSON, "text")
fParsedText <- fromJSON(fJSON_text, flatten = TRUE)
fDF <- as.data.frame(fParsedText)

return(fDF)
}
```

### Sample NHL Records Function Calls

``` r
#Example Calls Available for validation
gdfFranchiseAll<-getFranchiseInfo()
gdfFranchiseName<-getFranchiseInfo(aName="Devils")
gdfFranchiseNumber<-getFranchiseInfo(aFranchiseId = 2)
gdfFranchiseTeam<-getFranchiseInfo(aTeamId = 2)

gdfFranTotal<-getFranchiseTotals()
gdfFranTotalName<-getFranchiseTotals(aName="Devils")
gdfFranTotalNumber<-getFranchiseTotals(aFranchiseId = 2)
gdfFranTotalTeam<-getFranchiseTotals(aTeamId = 2)

gdfFranSeas<-getFranchiseSeasonRecords()
gdfFranSeasName<-getFranchiseSeasonRecords(aName="Devils")
gdfFranSeasNumber<-getFranchiseSeasonRecords(aFranchiseId = 2)
gdfFranSeasTeam<-getFranchiseSeasonRecords(aTeamId = 2)

gdfFranGoal<-getFranchiseGoalieRecords()
gdfFranGoalName<-getFranchiseGoalieRecords(aName="Devils")
gdfFranGoalNumber<-getFranchiseGoalieRecords(aFranchiseId = 2)
gdfFranGoalTeam<-getFranchiseGoalieRecords(aTeamId = 2)

gdfFranSkat<-getFranchiseSkaterRecords()
gdfFranSkatName<-getFranchiseSkaterRecords(aName="Devils")
gdfFranSkatNumber<-getFranchiseSkaterRecords(aFranchiseId = 2)
gdfFranSkatTeam<-getFranchiseSkaterRecords(aTeamId = 2)
```

## Functions to contact the NHL Stats API  

### NHL Stats API Modifier Helper  

Modifiers on the Stats API are less consistent. This function takes easy
hints in `aExpand` and turns them into the appropriate URL suffix.

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
```

### NHL Stats Pull

This API pulls information for a TeamID regarding Roster, schedules, and
some statistics. Note that most of these modifiers are pulled as
dataframe columns within the returned dataframe. This would be a good
opportunity for further cleanup and development.

This function has three arguments in addition to the common arguments of
the Records API queries.

`aExpand` can take the following values, pulling different summary
expansions:  
*“Roster”, “EZRoster”, “NextSched”,“PrevSched”,“Stats”, “TeamList”,
""*  
A value of TeamList needs valid teamIDs. The common arguments of `aName`
and `aNumber` supersede *“TeamList”*

`aSeason` is a string formatted Y111Y222 restricting the pull to the
season of interest. By default, an empty string will pull the current
season where applicable. Example: “20142015” to refer to the 2014-2015
hockey season.

`aTeams` is a comma separated string of teamIDs and requires the
‘TeamList’ expansion instruction. Example: “4,5,29” to pull the
RecordsAPI franchise pull mostRecentTeamIDs 4,5, and 29. Note that the
common inputs of `aName`, `aTeamId`, and `aFranchiseId` supersede this
field .

``` r
getTeamStats<- function(aExpand="", aSeason = "", aTeams = "", 
                        aName = "", aTeamId= "", aFranchiseId = "")
{
  #Process our inputs.  If any is non-empty, we need to add a query modifier.
  #Check for a common input
  fTeamStr = ""
  if(aName!="" | aTeamId != "" | aFranchiseId !="")
    {fTeamStr <- pullIDDecoder(aName, aTeamId, aFranchiseId)$TeamID}
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

  #Process results into a dataframe and return
  fRawJSON<- GET(fCallURL)
  fJSON_text<- content(fRawJSON, "text")
  fParsedText <- fromJSON(fJSON_text, flatten = TRUE)
  fDF <- as.data.frame(fParsedText)

  return(fDF)
}
```

### Sample calls for Stats API validation

``` r
#Available for validation
gdfTeamStatsa<- getTeamStats(aExpand = "Roster")
gdfTeamStatsb<- getTeamStats(aExpand = "EZRoster")
gdfTeamStatsc<- getTeamStats(aExpand = "NextSched")
gdfTeamStatsd<- getTeamStats(aExpand = "PrevSChed")
gdfTeamStatse<- getTeamStats(aExpand = "Stats")
gdfTeamStatsf<- getTeamStats(aExpand = "Teamlist", aTeams = "4,5,7")
gdfTeamStatsg<- getTeamStats(aSeason = "20142015")
gdfTeamStatsh<- getTeamStats(aExpand = "Stats", aSeason = "20142015")
gdfTeamStatsi<- getTeamStats(aName = "Devils")
gdfTeamStatsj<- getTeamStats(aFranchiseId = 2)
gdfTeamStatsk<- getTeamStats(aTeamId = 2)
gdfTeamStatsl<- getTeamStats(aTeamId = "2", aTeams = "4,5,7")

gdfTeamStatsm<-getTeamStats(aName="Devils")
gdfTeamStatsn<-getTeamStats(aFranchiseId = 2)
gdfTeamStatso<-getTeamStats(aTeamId = 2)
gdfTeamStatsp<-getTeamStats()
```

## Wrapper Function

### Wrapper Function to Combine Previous Endpoints

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
    print("aTeamId - MostRecentTeamID from the FranchiseInfo pull")
    print("aFranchiseId - id field from the FranchiseInfo pull.")
    print("Note 1- aFranchiseId is constant for franchises relocating or changing names")
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
```

### Wrapper Validation Calls

``` r
#Available for validation
gdfWrap<- pullNHLAPI()
gdfWrapa<- pullNHLAPI("FranchiseInfo")
gdfWrapb<- pullNHLAPI("FranchiseTotals")
gdfWrapc<- pullNHLAPI("FranchiseSeasonRecords")
gdfWrapd<- pullNHLAPI("FranchiseGoalieRecords")
gdfWrape<- pullNHLAPI("FranchiseSkaterRecords")
gdfWrapf<- pullNHLAPI("Team",aName = "Devils")
gdfWrapg<- pullNHLAPI("Team", aTeamId = 2)
gdfWraph<- pullNHLAPI("Team",aFranchiseId = "2")
gdfWrapi<- pullNHLAPI("Team")
```

# Analysis - Wayne Gretzky Performance

I don’t usually follow hockey, so this is the most that I have ever
looked at hockey statistics. From what I understand, it’s mostly about
scoring goals and checking people just hard enough that they don’t put
you in the penalty box. Wayne Gretzky was a household name growing up.
Let’s do some exploration for context about how good of a player he was.

## Initial Pull

Data is available in the FranchiseSkaterRecords endpoint:

``` r
#Pull Data from API
gTibGretz<- as_tibble(pullNHLAPI("FranchiseSkaterRecords"))

#Flag the records that are associated with Wayne Gretzky
gTibGretz <- gTibGretz  %>% mutate(Gretzky = ifelse(data.lastName=="Gretzky" & data.firstName == "Wayne",
         "Wayne Gretzky", "Rest of League"))
```

## Filter For Players with Several Games

First things first, there are probably a lot of alternates, rookies, and
other folks that didn’t get a lot of time on the ice in this pull. Let’s
take a look at the distribution of games played per season and filter
them out.

``` r
#Create a games per season column and plot the distribution
gTibGretz<- gTibGretz %>% mutate (GamesPerSeason = data.gamesPlayed / data.seasons)
g <- ggplot(data = gTibGretz, mapping = aes(x = GamesPerSeason))
g+ geom_histogram(binwidth = 1)+ggtitle("Games Per Season for Player Franchise Records")
```

![](README_files/figure-gfm/Low%20Playtime-1.png)<!-- -->

It’s a fairly uniform density, but it looks like a second mode starts
around 30 games. Let’s filter out entries with \<30 games per season

``` r
gTibGretz <- gTibGretz %>% filter(GamesPerSeason >= 30)
```

## Penalty Time vs Scoring

Ok, now let’s figure out how scoring goals correlates to penalty time.

``` r
#Scatterplot to evaluate bivariate relationship
g<- ggplot(data = gTibGretz, mapping = aes (x= data.mostPenaltyMinutesOneSeason, y = data.mostGoalsOneSeason))
g+geom_point(mapping = (aes(colour = data.positionCode, shape = Gretzky, size = Gretzky))) + 
  ggtitle("Personal Bests at a Franchise, Goals vs Penalty Time by Player, All Time")
```

![](README_files/figure-gfm/Penalty%20vs%20Scoring-1.png)<!-- -->

We see that Wayne had a clearly phenomenal tenure at one of his
franchises, apparently still being the all time scoring leader for goals
in a regular season by a significant margin across all franchises. He
was also apparently very good at not getting caught checking people, as
his maximum penalty time at that franchise was less than any other
center who scored more than 63 goals in one season.

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
6 hours of penalty time in a single season – I guess the coach didn’t
want to sit next to him on the bench. Dave Schultz, from – you guessed
it – Philadelphia.

Here’s the top 6 players for penalty times in a season:

``` r
kable(head(gTibGretz%>% select(data.firstName, data.lastName, data.franchiseName, data.mostPenaltyMinutesOneSeason,
            data.mostPenaltyMinutesSeasonIds)%>% arrange(desc((data.mostPenaltyMinutesOneSeason)))),
      format = "html",caption = "Top 6 Players - Penalty Minutes in 1 Season")
```

<table>

<caption>

Top 6 Players - Penalty Minutes in 1 Season

</caption>

<thead>

<tr>

<th style="text-align:left;">

data.firstName

</th>

<th style="text-align:left;">

data.lastName

</th>

<th style="text-align:left;">

data.franchiseName

</th>

<th style="text-align:right;">

data.mostPenaltyMinutesOneSeason

</th>

<th style="text-align:left;">

data.mostPenaltyMinutesSeasonIds

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

Dave

</td>

<td style="text-align:left;">

Schultz

</td>

<td style="text-align:left;">

Philadelphia Flyers

</td>

<td style="text-align:right;">

472

</td>

<td style="text-align:left;">

19741975

</td>

</tr>

<tr>

<td style="text-align:left;">

Paul

</td>

<td style="text-align:left;">

Baxter

</td>

<td style="text-align:left;">

Pittsburgh Penguins

</td>

<td style="text-align:right;">

409

</td>

<td style="text-align:left;">

19811982

</td>

</tr>

<tr>

<td style="text-align:left;">

Mike

</td>

<td style="text-align:left;">

Peluso

</td>

<td style="text-align:left;">

Chicago Blackhawks

</td>

<td style="text-align:right;">

408

</td>

<td style="text-align:left;">

19911992

</td>

</tr>

<tr>

<td style="text-align:left;">

Marty

</td>

<td style="text-align:left;">

McSorley

</td>

<td style="text-align:left;">

Los Angeles Kings

</td>

<td style="text-align:right;">

399

</td>

<td style="text-align:left;">

19921993

</td>

</tr>

<tr>

<td style="text-align:left;">

Bob

</td>

<td style="text-align:left;">

Probert

</td>

<td style="text-align:left;">

Detroit Red Wings

</td>

<td style="text-align:right;">

398

</td>

<td style="text-align:left;">

19871988

</td>

</tr>

<tr>

<td style="text-align:left;">

Basil

</td>

<td style="text-align:left;">

McRae

</td>

<td style="text-align:left;">

Dallas Stars

</td>

<td style="text-align:right;">

382

</td>

<td style="text-align:left;">

19871988

</td>

</tr>

</tbody>

</table>

## Performance vs Position Played

Alright, back to Wayne Gretzky. We just saw that he was a center. How
many skaters from different positions are we comparing against? There
are the offensive players (center, left wing, and right wing) We
wouldn’t want to compare that position against a defenseman for
scoring. Let’s generate average penalty time per game and and average
goals per game to see how the different positions compare.

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
kable(gTibPositionSummary, digits = 3,format = "html", caption = 'Performance Summary for Each Position')
```

<table>

<caption>

Performance Summary for Each Position

</caption>

<thead>

<tr>

<th style="text-align:left;">

data.positionCode

</th>

<th style="text-align:left;">

Gretzky

</th>

<th style="text-align:right;">

nPlayerFranchiseObs

</th>

<th style="text-align:right;">

avgGoalsPerGame

</th>

<th style="text-align:right;">

stdDevGoals

</th>

<th style="text-align:right;">

avgPenaltyBoxPerGame

</th>

<th style="text-align:right;">

stdDevPenaltyBoxPerGame

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

C

</td>

<td style="text-align:left;">

Rest of League

</td>

<td style="text-align:right;">

2200

</td>

<td style="text-align:right;">

0.191

</td>

<td style="text-align:right;">

0.106

</td>

<td style="text-align:right;">

0.572

</td>

<td style="text-align:right;">

0.454

</td>

</tr>

<tr>

<td style="text-align:left;">

C

</td>

<td style="text-align:left;">

Wayne Gretzky

</td>

<td style="text-align:right;">

3

</td>

<td style="text-align:right;">

0.513

</td>

<td style="text-align:right;">

0.301

</td>

<td style="text-align:right;">

0.367

</td>

<td style="text-align:right;">

0.086

</td>

</tr>

<tr>

<td style="text-align:left;">

D

</td>

<td style="text-align:left;">

Rest of League

</td>

<td style="text-align:right;">

2924

</td>

<td style="text-align:right;">

0.066

</td>

<td style="text-align:right;">

0.051

</td>

<td style="text-align:right;">

0.981

</td>

<td style="text-align:right;">

0.675

</td>

</tr>

<tr>

<td style="text-align:left;">

L

</td>

<td style="text-align:left;">

Rest of League

</td>

<td style="text-align:right;">

1821

</td>

<td style="text-align:right;">

0.189

</td>

<td style="text-align:right;">

0.110

</td>

<td style="text-align:right;">

0.889

</td>

<td style="text-align:right;">

0.840

</td>

</tr>

<tr>

<td style="text-align:left;">

R

</td>

<td style="text-align:left;">

Rest of League

</td>

<td style="text-align:right;">

1774

</td>

<td style="text-align:right;">

0.203

</td>

<td style="text-align:right;">

0.120

</td>

<td style="text-align:right;">

0.814

</td>

<td style="text-align:right;">

0.734

</td>

</tr>

</tbody>

</table>

From that quick summary, we see that Centers, Left Wingers, and Right
Wingers are all roughly consistent for number of points scored.
Defensemen understandably score far less. Just from the numerical
summary, we do see that Gretzky averages 3 standard deviations over the
league average for a center, quite impressive. The distribution of
penaltybox times is far noisier with a coefficient of variance close to
one. Gretzky is \~36% below average compared to the other Centers.

## Offensive Player Statistics

Let’s continue to look at where Gretzky falls relative to other
offensive players. Note that in the box plots below, we are adding some
jitter to the points. Goals, assists, and points in a single game are
all whole number values, we’re just adding some noise along the y-axis
to help visualize point density.

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

![](README_files/figure-gfm/boxplots-5.png)<!-- -->

Clearly an exemplary player with numbers that are matched by few across
his tenures at several franchises. Let’s see how his franchises
performed

``` r
#Figure out which teams Gretzky Played on, and filter data table down to those franchises
gTibGretzFranchList<- gTibGretz %>% group_by(Gretzky, data.franchiseId, data.franchiseName) %>%
  summarise(offensiveSkaterCount = n(), player.gamesPlayed = max(data.gamesPlayed))
```

    ## `summarise()` regrouping output by 'Gretzky', 'data.franchiseId' (override with `.groups` argument)

``` r
#Print the teams
kable(gTibGretzFranchList%>%select(-offensiveSkaterCount)%>%
        filter(Gretzky =="Wayne Gretzky"), format = "html", caption = "Teams Wayne Gretzky has Played For")
```

<table>

<caption>

Teams Wayne Gretzky has Played For

</caption>

<thead>

<tr>

<th style="text-align:left;">

Gretzky

</th>

<th style="text-align:right;">

data.franchiseId

</th>

<th style="text-align:left;">

data.franchiseName

</th>

<th style="text-align:right;">

player.gamesPlayed

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

Wayne Gretzky

</td>

<td style="text-align:right;">

10

</td>

<td style="text-align:left;">

New York Rangers

</td>

<td style="text-align:right;">

234

</td>

</tr>

<tr>

<td style="text-align:left;">

Wayne Gretzky

</td>

<td style="text-align:right;">

14

</td>

<td style="text-align:left;">

Los Angeles Kings

</td>

<td style="text-align:right;">

539

</td>

</tr>

<tr>

<td style="text-align:left;">

Wayne Gretzky

</td>

<td style="text-align:right;">

25

</td>

<td style="text-align:left;">

Edmonton Oilers

</td>

<td style="text-align:right;">

696

</td>

</tr>

</tbody>

</table>

``` r
#Join our Gretzky decoder to the franchise records
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
kable(gTibFranchSumm, digits = 1,format = "html",
      caption = "Average Franchise Records for Teams Gretzky Has Not (Rest of League) and Has (Wayne Gretzky) Played for")
```

<table>

<caption>

Average Franchise Records for Teams Gretzky Has Not (Rest of League) and
Has (Wayne Gretzky) Played for

</caption>

<thead>

<tr>

<th style="text-align:left;">

Gretzky

</th>

<th style="text-align:right;">

franchiseCount

</th>

<th style="text-align:right;">

averageMostGoals

</th>

<th style="text-align:right;">

stdDevMostGoals

</th>

<th style="text-align:right;">

averageMostWins

</th>

<th style="text-align:right;">

stdDevMostWins

</th>

<th style="text-align:right;">

averagePenaltyTime

</th>

<th style="text-align:right;">

stdDevPenaltyTime

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

Rest of League

</td>

<td style="text-align:right;">

37

</td>

<td style="text-align:right;">

303.7

</td>

<td style="text-align:right;">

86.2

</td>

<td style="text-align:right;">

48.4

</td>

<td style="text-align:right;">

11.8

</td>

<td style="text-align:right;">

1852.9

</td>

<td style="text-align:right;">

698.9

</td>

</tr>

<tr>

<td style="text-align:left;">

Wayne Gretzky

</td>

<td style="text-align:right;">

3

</td>

<td style="text-align:right;">

381.0

</td>

<td style="text-align:right;">

62.6

</td>

<td style="text-align:right;">

52.7

</td>

<td style="text-align:right;">

4.5

</td>

<td style="text-align:right;">

2147.0

</td>

<td style="text-align:right;">

115.2

</td>

</tr>

</tbody>

</table>

``` r
g<- ggplot(data = gTibGretzkyFranch, mapping = aes(x=Gretzky, color = Gretzky))

g+geom_boxplot(mapping = aes(y = data.mostGoals)) + ggtitle("Most Franchise Goals in a Season -- All Time") + 
  geom_jitter(mapping =aes(y=data.mostGoals)) 
```

![](README_files/figure-gfm/Analysis_2-1.png)<!-- -->

``` r
g+geom_boxplot(mapping = aes(y = data.mostWins)) + ggtitle("Most Franchise Wins in a Season -- All Time") + 
  geom_jitter(mapping =aes(y=data.mostWins)) + ylab('Most Wins')
```

![](README_files/figure-gfm/Analysis_2-2.png)<!-- -->

Implied in these plots is the idea that Wayne Gretzky was playing in his
franchises during all of their best years, which may be a stretch. We’ve
not actually limited to the seasons where Gretzky was playing for these
franchises – we’re looking at all time records. Still, I’d bet an
all-time leading scorer was on the roster for at least one of those
franchise goal records. For wins in a season, it looks like Gretzky’s
best team is in the top 7 while the others are more typical.

## Hockey Players Named Wayne

For one last look at his influence, let’s see how many skaters were
named Wayne in the 2000s, \~20 years after he debuted in the NHL.

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

kable(table(gTibWayne$NamedWayne, gTibWayne$After2000), format = "html",
      caption = "Players Named Wayne vs Year")
```

<table>

<caption>

Players Named Wayne vs Year

</caption>

<thead>

<tr>

<th style="text-align:left;">

</th>

<th style="text-align:right;">

Before 2000

</th>

<th style="text-align:right;">

Later than 2000

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

Not Wayne

</td>

<td style="text-align:right;">

10038

</td>

<td style="text-align:right;">

6759

</td>

</tr>

<tr>

<td style="text-align:left;">

Wayne

</td>

<td style="text-align:right;">

64

</td>

<td style="text-align:right;">

10

</td>

</tr>

</tbody>

</table>

Allowing some buffer for kids to grow up to be NHL age, the name Wayne
has not shot up in popularity in the years since Gretzky became famous.
Gretzky will have to settle for being that much more unique.

# Recap

This vignette defines several functions for the NHL API. More are
available based on the community documentation for the
[Records](https://gitlab.com/dword4/nhlapi/-/blob/master/records-api.md)
and [Stats](https://gitlab.com/dword4/nhlapi/-/blob/master/stats-api.md)
APIs. There is plenty of insight available from this simple if you have
a passing curiosity. There are also far more detailed endpoints to track
such things as player location throughout a game if you really want to
dive deeply into it.
