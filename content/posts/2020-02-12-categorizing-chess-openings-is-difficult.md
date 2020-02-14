---
template: post
title: The difficulties of classifying chess openings
slug: /posts/chesscat
draft: true
date: 2020-02-13T00:35:13.016Z
description: >-
  Many attempts have been made to categorize chess openings and their
  variations, none with complete success. I offer yet another proposal.
category: chess
tags:
  - chess
  - openings
  - ECO
  - Encyclopaedia of Chess Openings
---
_Note to self:  narrow the focus onto the disadvange of move sequences vs positions to ID openings.  Talk about FEN and rank-encoding.  Talk about NicBase vs acronym-string naming._ 

_Also, be sure to cover the case where new variations appear (TNs), possibly having more popularity when eventually named than some older obscure variation (do we reserve a special number, like '99x' to handle these? Or just rank them lower anyway)._

__

Chess openings, which are arrived at by initial moves of the chess pieces, are frustratingly hard to classify consistently. The first move by White is fairly easy to name: King's Pawn Opening, Queen's Pawn Opening, English, Reti...and on to rarer first moves. From there, though, it gets complicated quickly. Blacks responses to a King's Pawn opening are the Open Game (1...e5), the Sicilian Defense (1...c5), the French Defense (1...e6), the Caro-Kann (1...c6), and assorted others. The Open Game is transitory--it quickly leads to the more well-known Ruy Lopez, a Petroff, a Giuoco Piano, or who-knows-what.

Nevertheless, the names of openings and their variations and sub-variations are a type of classification, so long as they're consistent.  The opening **French Defense: Winawer, Delayed Exchange Variation, 4...Qxd5** can be converted to to a tree representation:

* French Defense
  * Winawer
    * Delayed Exchange Variation
      * 4...Qxd5

If this was done for all French Defense variations, you't wind up with a detailed, if verbose, classification of the French Defense and all its variations. This tree could be condensed into a single string. NicBase encodes openings with a two-letter acronym, followed by number codes for variations; the Winawer French is FR 8, but was not able to find a code for the Delayed Exchange Variation. Neither could I find a specific ECO code for this variation, only the generic French Winawer, without 4.e5 (C15).  Nor did it show up in SCID's codes,  where the closest I found was 







ECO

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

And opening notation isn't consistent!  Here's a screenshot from Chess.com's opening database:

![](/media/screenshot-2020-02-12-at-4.33.19-pm.png "Two notations, one move.")

The two most popular responses listed are 6...Ne7 and 6...Nge7.  They are the same move, but the opening database lists them as separate moves. Either notation is valid, is the problem. If instead this position was associated with an opening, there would be no ambiguity.

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

An identifier, be it FEN or this proposed rank-encoding, doesn't have to be easy to remember. It's an abstraction. A name can be associated with an identifier, much like Lawrence Smith is the name associated with the phone number +1(555)555-5555.  In fact, multiple names can be associated with an identifier: Larry Smith, Larry K. Smith, L. K. Smith, etc.  The names are what are interchangeably used to refer to the man, but the phone number is his and his alone. 

Just like with Larry though, an opening position can be referred to by multiple names. We don't care. What we do worry about, though, is that opening variation names can be hard to remember.  Some variations are not named with something easy to remember.  For instance, I have one variation in my database name:

French: KIA 2.d3 d5 3.Qe2

Well, you could remember that, but the deeper you go, the longer the name, the harder it gets. Still, names are useful things, and easer to remember than something like 3.9.12.4.1!

## Improving on the naive and radical

So one of the side-effects of using rank-encoding as an opening position identifier, is that patterns repeat. This gives us the opportunity to group together these repeating patterns and assign them a mnemonic abbreviation. Ruy Lopez (1.1.1.1.1) could be abbreviated 'RL'. The Morphy Defense would be RL.MD (RL.1); the Exchange Variation would be RL.MD.EV (RL.MD.2). You could use three-letter acronyms as mnemonics (QGD). 

In the case of French: KIA 2.d3 d5 3.Qe2, it might be encoded as FD.KIA.2 (3.Qe2 being the second most common move for White). 

A rule for naming opening rank ids would be: 

1. find the longest repeating segment
2. See if that segment (sequence of moves) has a name in the database; if so, abbreviate it and us it

2a) The segment 1.1.1.1.1.2 would have a description: Ruy Lopez: Morphy Defense, Exchange Variation, which would be abbreviated as shown earlier (RD.MD.EV).  

3. else, reduce the segment by one and try again. It is possible some variant of the Morphy Defence is wayyy down in the ranking: 1.1.1.1.1.12 and have no name. In that case, reduce the segment to search for by one, and wind up with RD.MD as the opening variation mnemonic.

## Can this process be automated?

I think so.  Let's try it with variations of the Bird Opening, because there aren't so many. 

<do it>

\*The exception being extensions of the encoding, as theory deepens and names/descriptions are assigned to these newer variations.
