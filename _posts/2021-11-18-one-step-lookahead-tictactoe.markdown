---
layout: post
title: "Bot plays Tic-Tac-Toe using One-step lookahead"
author: Rishabh Goel
categories: reinforcement-learning
---

Welcome Back! In this article we will design an agent that uses [one-step lookahead][oslookahead] to decide the best move. In the [previous article][article1], our agent played random moves which didn't turn out very successful with win rate of 59%(against another random agent).

This time around, I propose that we give our agent some intellect. What do I mean? Well, we will design the agent so that it looks at all the possible moves and then decides the best move.
This is much better than playing random moves, since it will play optimally at each step. However, we will look in the upcoming article that this is not the most best method.

## Import packages
Let's start by importing the necessary packages.

```python
import random
import numpy as np
from IPython.display import clear_output
from time import sleep
from util import *
```
util is a python file containing some of the code from previous article. You can find all the code [here](https://github.com/Coder-RG/blog-content/tictactoe).

## Let's create the battlefield

```python
ttt = np.zeros((3,3), np.uint16)
ttt
```
```
array([[0, 0, 0],
       [0, 0, 0],
       [0, 0, 0]], dtype=uint16)
```

Code for playing randomly was very simple. However, this time around we need to write a lot of lines.

## Create the necessary functions

All the functions contain docstring so as to help you understand their functionality.

```python
def get_valid_moves(ttt):
    """
    Get positions in the grid that are empty.
    
    Parameters:
    ===========
    ttt: numpy.ndarray
        --> Tic-Tac-Toe grid.
    
    Returns:
    ========
    valid_moves: list
        --> Grid positions which are empty.
    """
    grid = ttt.copy()
    if grid.shape != (9,):
        grid = grid.reshape(9)
    valid_moves = [x for x in range(9) if grid[x] == 0]
    return valid_moves
```

```python
def check_window(window, seq, player):
    """Count the player tiles in the concerned window and the remaining tiles
    should be empty. This will ensure that the player can play further in
    those tiles.
    
    For example,
    
         1 2 3  |  1 2 3
      1  X - X  |  - - X
      2  O - -  |  O - X
      3  O X O  |  O X -
    
    In the first grid for first column(X O O), if we were to check for this
    window, we can see that 'O' has better outcome with 2 of its pieces in
    a sequence, however, since there are no further tiles to play thus,
    should not be awarded high score.
    
    Moreover, in the second grid, the first column is much better to play
    a piece by the player "O", since there is an empty tile where further
    play is possible.
    
    Parameters:
    ===========
    window: list
        --> consecutive tiles to score
        
    seq: int
        --> number of same pieces to count
        
    player: int
        --> the player piece to count, eg: 1(X) or 2(O)
    
    Return:
    =======
    :boolean
    
    """
    return window.count(player) == seq and window.count(0) == 3-seq
```
```python
def count_windows(grid, seq, player):
    """
    Counts the number of windows that satisfy the given parameters,
    i.e. for the given player and the amount of pieces in a window.
    
    Parameters:
    ===========
    grid: numpy.ndarray
        --> 3x3 Tic Tac Toe grid
    
    seq: int
        --> number of same pieces to count
    
    player: int
        --> the player piece to count, eg: 1(Player 1) or 2(Player 2)
    
    Return:
    =======
    windows: int
        --> number of windows that satisfy the given criteria.
    """
    windows = 0
    # Horizontal(row)
    for row in grid:
        if check_window(list(row), seq, player):
            windows += 1
    
    # Vertical(column)
    for col in range(3):
        if check_window(list(grid[:,col]), seq, player):
            windows += 1
    
    # Diagonal
    window = list(grid[range(3), range(3)])
    if check_window(window, seq, player):
        windows += 1
    window = list(grid[range(2,-1,-1), range(2,-1,-1)])
    if check_window(window, seq, player):
        windows += 1
    return windows
```

## One step lookahead

The above code will help in implementing One-step lookahead but what exactly is it?

### Algorithm(Simplified)

>While game not done:
>    1. For each valid move in valid moves:
>         - play the move
>         - evaluate the play and assign a score
>    2. select all plays with score = highest score
>    3. If highest score is unique play that move
>    4. else choose randomly

### Move scoring
- +1000 if won i.e XXX or OOO
- +1 if next move will win the game i.e, XX or OO
- -100 if opponent will win in next move i.e it already has a sequence of 2.


```python
def get_heuristic(grid, player):
    """
    Evaluates move and assigns a score.
    
    Parameters:
    ===========
    grid: numpy.ndarray
        --> 3x3 Tic-Tac-Toe grid
    
    player: int
        --> 1(X) or 2(O) to denote the player
    
    Returns:
    ========
    score: int
        --> score the game board received
    """
    num_threes = count_windows(grid, 3, player)
    num_twos = count_windows(grid, 2, player)
    num_twos_opp = count_windows(grid, 2, player%2 + 1)
    score = 1e3*num_threes + num_twos - 1e2*num_twos_opp
    return score
```

```python
def score_move(grid, move, player):
    """
    Gets the score for a single move played.

    Parameters:
    ===========
    grid: numpy.ndarray
        --> 3x3 Tic-Tac-Toe grid

    move: int
        --> index where the play is performed. The grid is converted to a
            sequential array of shape (9,). Thus the move is a value between
            0...8
    """
    ttt = grid.copy()
    if ttt.shape != (9,):
        ttt = ttt.reshape(9)
    if ttt[move] != 0:
        raise "Invalid Move"
    ttt[move] = player
    return get_heuristic(ttt.reshape((3,3)), player)

def get_optimal_move(grid, player):
    """
    Return the best move to play out of all the possible moves.

    Parameters:
    ===========
    grid: numpy.ndarray
        --> 3x3 Tic-Tac-Toe grid
    
    player: int
        --> 1(X) or 2(O) to denote the player
    """
    valid_moves = get_valid_moves(grid)
    scores = []
    for valid_move in valid_moves:
        scores.append(score_move(grid, valid_move, player))
    max_score = max(scores)
    max_score_indices = [index for index,score in enumerate(scores) if score == max_score]
    return valid_moves[random.choice(max_score_indices)]
```

## Putting it all together

First, our new agent will compete against random agent. Fingers crossed!

### Against Random Agent

```python
agent1 = {'piece':1}
agent2 = {'piece':2}

result = {'agent1':0, 'agent2':0, 'draws':0, 'total_games':0}
ngames = 100

for _ in range(ngames):
    # Reset the grid
    ttt = np.zeros((3,3), np.uint16)
    result['total_games'] += 1
    for iteration in range(9):
        # Optimal Agent
        if iteration%2 == 0:
            move = get_optimal_move(ttt, agent1['piece'])
            ttt.reshape(9)[move] = agent1['piece']
        # Dumb Agent
        else:
            move = get_random_tile(ttt)
            ttt[move] = agent2['piece']
        winner = check_win(ttt)
        clear_output()
        if winner is not None:
            if winner == 1:
                result['agent1'] += 1
            else:
                result['agent2'] += 1
            break
    if winner is None:
        result['draws'] += 1
print_result(result)
```

```
                          Stats                       
    --------------------------------------------------
    Total Games:100   Agent1:98    Agent2:1     Draws:1 
```

Woohoo! Our agent won 98-1 against random agent. This is a significant improvement. Now let's compete lookahead agent with a copy of itself.

### Against replica of self
```python
agent1 = {'piece':1}
agent2 = {'piece':2}

result = {'agent1':0, 'agent2':0, 'draws':0, 'total_games':0}
ngames = 100

for _ in range(ngames):
    ttt = np.zeros((3,3), np.uint16)
    result['total_games'] += 1
    for iteration in range(9):
        if iteration%2 == 0:
            move = get_optimal_move(ttt, agent1['piece'])
            ttt.reshape(9)[move] = agent1['piece']
        else:
            move = get_optimal_move(ttt, agent2['piece'])
            ttt.reshape(9)[move] = agent2['piece']
        winner = check_win(ttt)
        if winner is not None:
            if winner == 1:
                result['agent1'] += 1
            else:
                result['agent2'] += 1
            break
    if winner is None:
        result['draws'] += 1
print_result(result)
```
```
                          Stats                       
    --------------------------------------------------
    Total Games:100   Agent1:30    Agent2:24    Draws:46
```

Fascinating. There are more draws than either agent's wins.

This brings an end to our one-step lookahead agent build. I hope this interested you to look out for the next article where we will implement the n-step lookahead agent.


[article1]: {{site.baseurl}}/{% post_url 2021-11-16-random-tictactoe %}
[oslookahead]: https://www.kaggle.com/alexisbcook/one-step-lookahead