---
template: post
title: Visualizing Grandmaster Openings
slug: /posts/hmgm
draft: false
date: 2020-02-11T13:16:01.191Z
description: >-
  Data visualization is a process by which graphics are used to enhance
  statistics. This can be applied to data from chess games.
category: Heat Maps
tags:
  - heat map
  - chess
---
# What is data visualization?

Anybody who has used a diagram or a chart to illustrate numeric or other data has practiced [data visualization](https://www.lexico.com/definition/data_visualization). This post will be about a specific data visualization technique called a [heatmap](https://www.lexico.com/definition/heat_map). 

A heatmap uses colors and location to characterize data. That data may be about [actual heat](https://www.weathercentral.com/weather/us/maps/current_temperatures.html) (temperature), but its application as a data visualization tool [is not limited](https://www.businessinsider.com/where-tornadoes-strike-the-us-most-often-2014-4) to a specific type of data. 

![Heatmap showing tornado frequency in the U.S.](/media/tornado.jpeg "Heatmap showing tornado frequency in the U.S.")

## Example: a heat map of poker odds

My first practical application of a heatmap was to [visualize poker odds in Texas Hold'em](https://pokermap.netlify.com/).  In this poker variant, players are dealt two hole cards facedown at the start of the hand.  What are the odds that any two hole cards will ultimately win the hand?  There are variables that affect the odds, such as number of players and whether the hole cards are of the same suit.  With five players and the suited hole cards, here are the odds in heatmap form:

![Poker odds of suited hole cards, five players](/media/screenshot-2020-02-11-at-6.27.21-am.png "Poker odds of suited hole cards, five players")

# Visualization of chess game data

It is possible to use heatmaps to visualize chess-related data, such as a grandmaster's opening repertoire. To do just that, I downloaded PGN files and parsed each game score. For the first 18 moves, I generated a FEN string that encode the position of the pieces after the move was played. 

While an opening can be thought of a sequence of moves, in this case each position after a move is played is associated with an opening in an opening database (if it exists). This is done for the first 18 full moves of each game.  For instance, the move sequences:

1.d4 d5 2.c4...

and

1.c4 d5!? 2.d4...

would result in the same position, and that position is identified as the Queen's Gambit.  It doesn't matter that the latter set of moves started off as an English. Each new move leads to a new position and in turn are associated with a opening variation if one exists.

## The games of Shirov and Morozevich

A huge amount of data isn't needed to test idea of using heatmaps to visualize game openings, so I started with[ two pgn files](https://chessproblem.my-free-games.com/chess/games/Download-PGN.php) that contained games of Grandmasters Alexei Shirov and Alexander Morozevich, about 1,000 total. These games where played between 1990 and 2006.  For an opening book, I based mine on an extended [ECO](https://en.wikipedia.org/wiki/Encyclopaedia_of_Chess_Openings) encoding called [SCID](http://watfordchessclub.org/images/downloads/scid.eco). 

The data generated from the games and openings are stored on a database "in the cloud". That data is then used as a basis for the heatmaps I'm about to show.

# Mapping frequency of occurrence

How often do these two grandmasters encountered any position in their game openings? Positions from early moves will be encountered more frequently, later moves will lead to reoccurring positions as well. The following heatmaps only show data for positions that occur at least 10 times in the game data for each grandmaster.

## GM Shirov's openings

A player can be either White or Black. To begin, let's map Shirov's openings as the White player. 

![Shirov's openings as White](/media/screenshot-2020-02-11-at-8.04.05-am.png "Shirov's openings as White")

The first thing to notice is that Shirov during this period played either e4 (B00, 394 times) or much less often, d4 (A40, 35 times).  Red indicates a higher occurrence of a position and blue the least number of occurrences.

### Expanding the map

If you're like me, you aren't able to remember what opening and ECO code refers to, so let's expand the heat map:

![expanded heat map](/media/screenshot-2020-02-11-at-8.12.09-am.png "expanded heat map")

This visualization takes up more screen real estate, but you don't have to remember that, say, C00 is the French Defence.

## GM Morozevich's openings

By using the condensed maps, one can easily compare the opening repertoire of GM Morozevich vs GM Shirov; here's Morozevich's heat map:

![heat map for Morozevich](/media/screenshot-2020-02-11-at-8.17.49-am.png "heat map for Morozevich")

Right away we see that Morozevich mixes it up just a little by throwing in the occasional English Opening (A10). Also, the second row (move 2) shows two COO (French Defence) squares, whereas Shirov only had one.  What gives?  Well, ECO codes are not that specific, and the same code can encompass two variations. By expanding the map, it can be seen what the difference is:

![Two variations of the French](/media/screenshot-2020-02-11-at-8.25.11-am.png "Two variations of the French")

Ah!  Morozevich occasionally likes to respond 2.d3 instead of 2.d4, whereas Shirov consistently plays the 2.d4.

## Mapping frequence of responses

It's possible to drill down a half-ply further and look at the responses to a move. Here are the opponent's responses that Morozevich encountered after 2.d4 in the French:

![Responses to 2.d4](/media/screenshot-2020-02-12-at-11.40.19-am.png "Responses to 2.d4")

All but two of Moro's opponents chose to go into a French Defence proper with a response of 2...d5, but in one case the opening transposed to an Owen Defense, and  the other to a Franco-Benoni variation of the French.

## Mapping frequence of success

One last thing to be examined is how often an opening leads to a successful outcome. Again looking at the openings of Shirov, I can switch the map to color-code each opening position by average score:

![Average score of games where opening position occurs](/media/screenshot-2020-02-11-at-8.40.50-am.png "Average score of games where opening position occurs")

It at first seems curious that Shirov scored higher, on average, with opening positions he encountered less frequently (those to the right and bottom). Why is that? Wouldn't he want to play an opening that scored better more often?

There are several possible explanations.  B07, for example, is the Pirc, hand that could be a harder opening to defend as Black than C60, which is a Ruy Lopez.  Few occurrence of a position also means that any single game's outcome has a greater effect on the average score.

# Conclusion

Some interesting chess-related visualizations can be done with heatmaps.  I hope to explore more ideas in future posts.
