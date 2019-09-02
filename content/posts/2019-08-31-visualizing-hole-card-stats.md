---
template: post
title: Visualizing Hole Card Stats
slug: /posts/holecardmap
draft: true
date: 2019-08-31T15:48:23.813Z
description: >-
  With a heat map, it is possible to look at Texas Holdem pre-flop hole card
  winning percentages and see them in a new light.
category: Heat Maps
tags:
  - heat map
---
You are probably familiar with [Texas Holdem](https://en.wikipedia.org/wiki/Texas_hold_%27em), a variant of poker where two cards, known as hole cards, are dealt face down to each player, and then five community cards are dealt face up in three stages, for a total of four stages in all.  At each stage, players have the opportunity to bet or raise.

If you're dealt two hole cards before the flop, what are your chances of winning? If you know, you can bet accordingly (although it should be said that there are another of other factors that influence betting, including judging whether any opponent is bluffing or not). 

Professor Apu Kapadia took a statistical approach and ran 4 billion simulated of poker games to determine the outcomes.  The results are lists of winning percentages based on number of players, from two to ten. And while each list is very informative, there are 169 rows of data in each of them.  It's hard to see a pattern in numbers, other than showing high ranking hole cards do better than lower ranking hole cards.

I wrote a simple heat map application which I wrote a bit about for freeCodeCamp. Looking for a practical use, I ran across Dr. Kapadia's statistics and they were perfect for this demonstration.

## Massaging the statistics

The statistic are in HTML; I had two options:

1. screen-scrape the data from the Document Object Model using Cheerio.  
2. try to cut and paste the stats on the rendered HTML page into a file, then convert that to JSON format using some regex magic.

Since the stats bore a copyright notice, I asked which method the Professor would prefer, and he was okay with option 2.  The regex wasn't too hard to write, and so I wound up with JSON like the following:

<gist here>

This still wasn't in the proper format for the head map API, but few lines of code took care of that

<gist here>

Now all was ready.

## The heat map API

The jsheatmap package was written in TypeScript, and takes the following constructor parameters:

<gist here>

When the method getData() is called, what is returned is a data structure that shows how the raw input data was scaled, and what the corresponding color gradient will be for each scaled value.

<gist here>

Now all that is left is to present the map data in a pleasing format.

## The React application

I alway state up front that I am not a web designer (on account of an aesthetics disability) so in this case I will use an HTML table to display there results.

The first thing I want to do is to add some user interactivity by way of radio buttons and check boxes. There are three things the user can specify:

1. number of players
2. suited or unsuited hole cards
3. whether to include ties

I talked about the effect of number of players before. If hole cards are suited (both are hearts, for example), then winning odds increase a small amount. Ties are when pots are split because two or more players have the same rank of cards in their 5-card hand (their two hole cards plus three of the five community cards).

Several screenshots follow:

`<screnshots here>`

## What the heat map makes noticeable

More players lessen the odds

Low ranking pairs don't help much

Suited hole cards have more of an effect when there are more players

## Conclusion

I think this post has demonstrated that even a simple heat map can be useful for finding patterns and correlations in large amounts of data. Feel free to explore the source code for both **jsheatmap** and **pokermap**.
