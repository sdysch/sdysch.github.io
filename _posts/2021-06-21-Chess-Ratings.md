---
layout: post
title: Chess.com game and puzzle ratings
categories: [Miscellaneous]
---

## Chess.com ratings

After a ~20 year hiatus of playing chess, I started again for fun (and to cope with the lockdown boredom), on the popular website [chess.com](https://www.chess.com/).
I was curious how the ratings of [puzzles](https://www.chess.com/puzzles/rated) and the different types of games would correlate, particularly because my peak (at the time of writing)
puzzle and rapid ratings (1822 and 1061, respectively) were very different.
Looking for something to do on a rainy afternoon, I discovered the chess.com has [an API](https://www.chess.com/news/view/published-data-api), that would allow me to explore this.
There is also the handy [python wrapper](https://pypi.org/project/chess.com/) that can be used in conjunction with [requests](https://pypi.org/project/requests/).

There were a couple of caveats and assumptions that I had to make:
* Free users can only do a certain number of rated puzzles/day, whereas the number of rated games is unlimited
* One can only access a list of players by country, not a complete list of players. I only chose the UK
	* This would anyway take _far too long_ to run, because many requests would have to be made
* New accounts will skew the results, so I only chose accounts which had been active for at least one month
* There are many different chess categories one can play in
	* I chose the highest score of the _bullet_, _rapid_, and _blitz_ categories, requiring that at least 10 games had been played in each category
	* This removes any bias from the default ratings assigned to newer players
* I could not find a way to access the number of puzzles each player had solved, correctly or incorrectly, so I required players to have at least solved 1 puzzle (there is no rating otherwise)

I have very little experience with using _requests_, so possibly I did something wrong.
One has to make a new request for each player, and it takes a _very_ long time to do so.
Because there are >10000 active players in the UK, I limited to 7500 players.
The code is available [here](https://github.com/sdysch/chess_puzzle_vs_game_ratings).
Any suggestions for optimisation are very welcome!

Here is the scatter plot showing the correlation (Pearson correlation coefficient of 0.57) of puzzle and game scores (chosen as described above).
The red line shows the case of equal puzzle and game ratings.


![](/images/210621/chess_game_ratings_vs_puzzles.png)

It seems that indeed the puzzle rating tends to be higher than the game rating. I'm not sure what the horizontal "bands" on the plot correspond to, but I suspect that these correspond to some default ratings where a player has only played a handful of puzzles.
