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

I wrote a simple library to generate data for a heat map, given a set of raw data. My first practical application of it was to visualize poker odds in Texas Hold'em.  In this poker variant, players are dealt two hole cards at the start of the hand.  What are the odds that any two hole cards will ultimately win the hand?  There are some variables that affect the odds, such as number of players, and whether the hole cards are of the same suit. As an example, here's the visualization of odds when there are five players and the hole cards are suited:

![Poker odds of suited hole cards, five players](/media/screenshot-2020-02-11-at-6.27.21-am.png "Poker odds of suited hole cards, five players")

chess openings

Introduction to the application

The games of Shirov and Moresevich 

Openings identified by position, not move sequence

Mapping frequency of occurrence

Expanding the map

Mapping frequence of responses

Mapping frequence of success

interpretation

Conclusion
