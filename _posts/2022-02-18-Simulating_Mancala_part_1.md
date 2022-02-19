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

<img src="{{ site.baseurl }}/images/mancala/mancala.jpg" height=300 alt="Mancala board" figcaption="Mancala board" class="center"/>

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
* When the game is over, any remaining marbles on each player's side are added to their total

The winner is the player with the most marbles.

```python
from core.game.Board import Board
board = Board(n_start_marbles=4)
print(board)

'''
        4 4 4 4 4 4
[0]                      [0]
        4 4 4 4 4 4

'''

board.make_player_move(1, 3, 1) # makes a move for player 1, from bucket 3, on side 1
print(board)

'''
        4 4 4 4 4 4
[0]                      [1]
        4 4 0 5 5 5
'''

```

### Initial analysis
Throughout the rest of this article, _player one_ refers to the player who moves first, and _player two_ is the player who moves second.
Initially, I chose to simulate both players as a random agent. That is, from the available moves a player has, one is selected randomly with equal weightings per move.
As we will later see, this does not form an optimal strategy for either player.

For these tests, I ran 10000 simulations with player one and player two utilising a random strategy for their move choice.
![Distribution of player moves and scores for the random-random strategy pairing]({{ site.baseurl }}/images/mancala/random_random_nmoves_score.png)

It's clear that player one tends to have a higher score than player two, and thus we would expect player one to have a higher win percentage.
Player one also makes more moves than player two, which is to be expected with the higher overall score that player one has.
Examining the distribution of player moves for the three different outcomes (win, lose, draw) provides confirmation of this.
![Distribution of player moves for different game outcomes]({{ site.baseurl }}/images/mancala/random_random_move_distributions.png)
These distributions are unit normalised, to better compare the shapes.

When players win, the mean number of moves made is higher than the mean number of moves made when they lose.
This might suggest that a selection strategy which maximises the number of moves made on a given turn could perform better than a random choice.

Because player one always moves first, and gets a free choice, it is interesting to calculate the win, lose, and draw rates for the different first move choices.
![First move win rates for random-random strategy]({{ site.baseurl }}/images/mancala/random_random_first_move_prob.png)

### Different strategies
From the initial analysis, we know that players tend to win if they choose buckets which maximise the number of moves they make.
Clearly, making choices which maximise their score increase will also perform well.
With these ideas in mind, we can develop different strategies for players to apply, and then simulate them against each other.

**Exact strategy**

For this strategy, the player makes choices by favouring moves which will end in the player's goal, therefore giving them another move.
If multiple moves of this kind are possible, they are played in order, starting with the closest to the player's goal, enabling a repeated advantage of this rule.

```python
from core.game.Board import Board
board = Board()
board.player_one_cups = [1, 3, 0, 3, 2, 1]
print(board)

'''
        6 6 6 6 6 6
[0]                      [0]
        1 3 0 3 2 1
'''

board.make_player_move(1, 6, 1)
board.make_player_move(1, 5, 1)
board.make_player_move(1, 4, 1)
print(board)

'''
        6 6 6 6 6 6
[0]                      [3]
        1 3 0 0 1 2
'''
```

If no such moves are possible, then the strategy falls back to either random, or a new strategy described below.

**Max score strategy**

For this strategy, the player chooses the bucket which will give them the largest score increase
