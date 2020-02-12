---
template: post
title: Visualizing Grandmaster Openings
slug: /posts/hmgm
draft: true
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

Anybody who has used a diagram or a chart to illustrate numeric or other data has practice [data visualization](https://www.lexico.com/definition/data_visualization). This article will be about a specific data visualization technique called a [heat map](https://www.lexico.com/definition/heat_map). 

## What is a heat map?

A heat map uses colors and location to illustrate data. That data may be about [actual heat](https://www.weathercentral.com/weather/us/maps/current_temperatures.html), a.k.a. temperature, but their application as a data visualization tool [is not limited](https://www.businessinsider.com/where-tornadoes-strike-the-us-most-often-2014-4) to a specific type of data. 

![Heatmap showing tornado frequency in the U.S.](/media/tornado.jpeg "Heatmap showing tornado frequency in the U.S.")

## A heat map of poker odds

I wrote a simple library to generate data for a heat map, given a set of raw data. My first practical application of it was to [visualize poker odds in Texas Hold'em](https://pokermap.netlify.com/).  In this poker variant, players are dealt two hole cards at the start of the hand.  What are the odds that any two hole cards will ultimately win the hand?  There are some variables that affect the odds, such as number of players, and whether the hole cards are of the same suit. As an example, here's the visualization of odds when there are five players and the hole cards are suited:

![Poker odds of suited hole cards, five players](/media/screenshot-2020-02-11-at-6.27.21-am.png "Poker odds of suited hole cards, five players")

# Visualization of chess game data

I've just started exploring the use of heat maps to visualize data related to chess. My initial effort has been to see how to apply them to grandmaster games.

## Introduction to the application

To start, I downloaded pgn files and parsed each game to extract FEN position data for the first 18 moves. Whereas most people would think of an opening as a sequence of moves, here I am associating each position with an opening in an opening database. This eliminates the need to handle transitions from one opening to another.  For instance, the move sequences:

1.d4 d5 2.c4...

and

1.c4 d5!? 2.d4...

would result in the same position, and that position is identified as the Queen's Gambit.  As more moves are made,  new positions arise, and those in turn are associated with a opening variations (if one is found for the position).

## The games of Shirov and Morozevich

I didn't need a huge amount of data to test this idea of using heat maps to visualize game openings, so I started with[ two pgn files](https://chessproblem.my-free-games.com/chess/games/Download-PGN.php) that contained roughly contemporaneous games of Grandmasters Alexei Shirov and Alexander Morozevich, about 1,000 total. These game where played between 1990 and 2006.  For an opening book, I based mine on an extended ECO encoding called [SCID](http://watfordchessclub.org/images/downloads/scid.eco). 

Both games and openings were stored "in the cloud" after parsing and enhancement.

# Mapping frequency of occurrence

My first idea was to visualize how often either grandmaster encountered a position in their openings. By the nature of the game, the first moves will be encountered more frequently, but we can also see how often later move sequences lead to a position.  In the following heat maps, I will be limiting the data to include only those opening positions that are encountered at least 10 times.

## GM Shirov's openings

A player can be either White or Black. To begin with, let's map Shirov's openings as the White player. 

![Shirov's openings as White](/media/screenshot-2020-02-11-at-8.04.05-am.png "Shirov's openings as White")

The first thing to notice is that Shirov during this period played either e4 (B00, 394 times) or much less often, d4 (A40, 35 times).  Note that red indicate increasing frequence of the position in Shirov's games, and blue represents few occurrences of a position.

### Expanding the map

If you're like me, you aren't able to remember what opening and ECO code refers to, so let's expand the heat map:

![expanded heat map](/media/screenshot-2020-02-11-at-8.12.09-am.png "expanded heat map")

This takes up a lot more screen real estate, but packs in more information.

## GM Morozevich's openings

Using the condensed map, one can easily compare the opening repertoire of GM Morozevich vs GM Shirov; here's Morozevich's heat map:

![heat map for Morozevich](/media/screenshot-2020-02-11-at-8.17.49-am.png "heat map for Morozevich")

Right away we see that Morozevich mixes it up just a little by throwing in the occasional English Opening (A10). Also, the second move row show's two COO (French Defence) squares, whereas Shirov only had one.  What gives?  Well, ECO codes are often not very specific, and the same code can encompass two variations. By expanding the map, it can be seen what the difference is:

![Two variations of the French](/media/screenshot-2020-02-11-at-8.25.11-am.png "Two variations of the French")

Ah!  Morozevich occasionally likes to respond 2.d3 instead of 2.d4, whereas Shirov consistently plays the latter.

## Mapping frequence of responses

It's possible to drill down further and look at the responses to a move; let's look at the responses when Moro plays 2.d4 in response to the French:

Here we can see that Moro's opponent's usually go into a French Defence with 2...d5, but occasionally transpose to a 2...b6 variation, or a variation known as the Franco-Sicilian (2...c5).

## Mapping frequence of success

One last thing to be examined is how often an opening leads to a successful outcome? Back to Shirov, I can switch the map to color-coding the frequency of occurrence of a position to instead having the map indicate the average score of the games where the opening position arises earlier on:

![Average score of games where opening position occurs](/media/screenshot-2020-02-11-at-8.40.50-am.png "Average score of games where opening position occurs")

So, curiously, Shirov scored higher, on average, with opening positions he encountered less frequently (these boxes are still ordered by frequency--it's the colors that have changed). Why is that? Wouldn't he want to play an opening that scored better more often?

There are several possible explanations.  B07, which is a Pirc, could be a harder opening to play than C60, which is a Ruy Lopez (at least in my experience).  It may also be that since he plays 2.d4 much more rarely, his opponents are either less prepared or of a much lower rating on average. And it good be just luck, which plays a bigger role when few games are played with 1.d4.

# Conclusion

I've shown some of the interesting chess-related visualizations that can be done with heat maps.  I hope to explore more visualization ideas in future posts.
