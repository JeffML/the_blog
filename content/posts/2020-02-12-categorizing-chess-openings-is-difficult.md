---
template: post
title: Categorizing chess openings is difficult
slug: /posts/chesscat
draft: true
date: 2020-02-13T00:35:13.016Z
description: >-
  Various attempts have been made to categorize chess openings, all with
  limitations. Can a more complete an systematic approach lead to better
  results?
category: chess
tags:
  - chess
  - openings
  - ECO
  - Encyclopaedia of Chess Openings
---




I find the Encyclopaedia of Chess Openings (ECO) classification of openings wanting. In part this is because I'm no longer a serious chess player, and don't often research openings by ECO code. More often, I am looking up an opening based on a chess position, or by drilling down through moves in an openings database.

ECO was devised to cover serious opening lines--those played by masters in tournament games. If it's a seldom-seen opening, then it will get lumped under one of the 00 classifications: A00, B00, C00, D00, and E00. Note that early opening moves also fall under 00 classification: the French Defense is C00, and that encompassed positions like this:

![French: Normal Variation](/media/screenshot-2020-02-12-at-4.53.48-pm.png "French: Normal Variation")

But also the less common:

![](/media/screenshot-2020-02-12-at-4.53.24-pm.png "French: King's Indian Attack")

Now, truthfully, the C00 classification doesn't really include the main lines of the French.  If I look up the classification in an old Chess Informant I have, I see symbology that indicates C00 encompasses French Defense openings _without_ 2.d4, or with 2.d4, but without 2...d5, or with 2...d5 but without 3.Nc3:

![](/media/20200212_170909-edited.jpg "From Chess Informant, #66")

This is workable for humans that read, but it doesn't really work for classifying arbitrary positions.

The point is, though, that ECO breaks down some openings (Ruy Lopez, C60-C99) into fine detail, and others (Scotch Game, C44-C45) not so much. Even the Ruy, with its 39 classifications, does not cover many of the possible variations employed in master games.

# What are the alternatives?

Given that the ECO codes are too general and hard to remember, what alternatives are there?  In practice, specific opening variations are identified by ECO plus a description, but that can introduce arbitrariness of its own. Names for general openings are not even standardized:  [is it Petrov's Defense, Petrov Defence, Petroff's Defence, Petrov's Game, Russian Defence, or Russian Game](https://en.wikipedia.org/wiki/Petrov%27s_Defence)? 

One more thing:  ECO classifications are base on move sequences, so the same position arising from different move sequence may have different classifications.  1.c4 is English; 1...d5 in response is the Anglo-Scandinavian;  now if White plays 2.d4,  is it still a variation of the English?  Looks like a Queen's Gambit to me.

Positions should identify openings. A position might be have many opening variation names, but if it is identified solely by position, there is no ambiguity. One could use FEN notation as an identifier. The drawback, of course, is that it is even less memorable than ECO code.

The other drawback of using FEN notations ids for opening variations is that it is not easy to determine if any two FENs arise from the same opening. It could probably be done, but it wouldn't be simple.

Remember, too, that ECO attempted to classify openings in a way that reflected their popularity among masters. FENs don't give you any clue as to how often that opening variant occurs. What to do?

## Try something naive and radical

Let's say there exists a large game database that has all the master games played from 1950 to 2019 (I'm sure something like it exists). Now, encode each opening and opening variation by its polarity. If I were to pick the five most common moves in Chess.com's database (two plies per move), I would get:

<moves here>

Since ply represents the most popular move within the most popular variation, we can encode it as follows:

1.1.1.1.1.1.1.1.1.1

Crazy idea, right?  How is this possibly better than the corresponding ECO code of <eco code here>? Well, it's not! It's worse! 

It is however and identifier of a specific position.  It is true, another opening database might have a different ranking of popularity, but we only need to use one really complete database for the initial encoding. We don't care if the ranking change between databases or over time.  Once the encoding is done, it's done.*

An identifier, be it FEN or this proposed rank-encoding, doesn't have to be easy to remember. It's an abstraction.





\*The exception being extensions of the encoding, as theory deepens and names/descriptions are assigned to these newer variations.