---
layout: post
title: Simulating Mancala strategies with python
categories: [Simulations, Miscellaneous]
fontsize: 3em
---

## Simulating Mancala, part 1
_The code used for all simulations and analysis described in this post can be found on my [github](https://github.com/sdysch/mancala)_

A few years ago, my partner introduced me to the board game [mancala](https://en.wikipedia.org/wiki/Mancala), also known as Kalaha.
I have always suspected that there is an advantage to whoever moves first. Let's see if we can test this hypothesis with python!

[<img src="{{ site.baseurl }}/images/mancala/mancala.jpg" height=300 alt="Mancala board" figcaption="Mancala board" class="center"/>]({{ site.baseurl }}/)

### Rules
Mancala is a two-player board game, where players compete to capture more marbles than their opponent.
The rules are as follows:

* Each player has six buckets, initially containing four marbles. They also have a score bucket.
* A move is made by player one choosing one of their six buckets, collecting all their marbles.
* The player then distributes the marbles in subsequent buckets, including the player's score bucket, moving in a anti-clockwise direction until no marbles remain.
* The opponent's score bucket is skipped over.

Depending on the bucket that a player places their last marble in, additional rules are applied:
* If the player ends in a non-zero bucket, then they collect the stones in that bucket and continue moving anti-clockwise around the board.
* If the player ends their turn in their score bucket, then they get to choose a new move from their non-zero buckets.
* If the player ends in an empty bucket, then the player's turn is over.
* The game is over when either half of the marbles have been captured, or one player manages to empty all the buckets on their side.
* Any remaining marbles on each player's side are added to their total. The winner is the player with the most marbles
