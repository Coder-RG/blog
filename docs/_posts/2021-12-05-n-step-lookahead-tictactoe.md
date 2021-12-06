---
layout: post
title: "Bot plays Tic-Tac-To using N-step lookahead"
categories: reinforcement-learning
---

Good to see you! In this article we will design a bot which can play Tic-Tac-Toe
using [N-Step Lookahead][nstepwiki], a.k.a minimax. In the [previous article][article2], we created a
bot which played using one-step lookahead. However, this time our bot will look
ahead much further into the future rather than just one-step.

## Import packages
```python
import numpy as np
import random
from util import *
```

## Create the battlefield
```python
ttt = np.zeros((3,3), np.uint16)
ttt
```
```
array([[0, 0, 0],
       [0, 0, 0],
       [0, 0, 0]], dtype=uint16)
```

## Code functions
```python
def get_valid_moves(grid):
    valid_moves = []
    for row in range(3):
        for col in range(3):
            if grid[row,col] == 0:
                valid_moves.append((row,col))
    return valid_moves
```
```python
def check_win_draw(ttt):
    grid = ttt.copy()
    draw = True
    for row in grid:
        if list(row).count(0) > 0:
            draw = False
            break
    # check rows
    for row in range(3):
        if list(ttt[row,:]).count(1) == 3:
            return 1
        elif list(ttt[row,:]).count(2) == 3:
            return 2
    # check columns
    for col in range(3):
        if list(ttt[:,col]).count(1) == 3:
            return 1
        elif list(ttt[:,col]).count(2) == 3:
            return 2
    # Check diagonals
    if list(ttt[range(3),range(3)]).count(1) == 3 or list(ttt[range(3),range(2,-1,-1)]).count(1) == 3:
        return 1
    elif list(ttt[range(3),range(3)]).count(2) == 3 or list(ttt[range(3),range(2,-1,-1)]).count(2) == 3:
        return 2
    return 0 if draw else -1
```

## N-Step Lookahead

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
    num_three = count_windows(grid, 3, player)
    num_twos = count_windows(grid, 2, player)
    num_twos_opp = count_windows(grid, 2, player%2 + 1)
    num_three_opp = count_windows(grid, 3, player%2 + 1)
    score = 1e5*num_three + num_twos - 1e2*num_twos_opp - 1e3*num_three_opp
    return score
```


```python
def minimax(grid, depth, player, maximising):
    if depth == 0 or check_win_draw(grid) != -1:
        heuristic = get_heuristic(grid,player)
        return heuristic
    ttt = grid.copy()
    if maximising:
        value = -0b1<<20
        for valid_move in get_valid_moves(grid):
            ttt[valid_move] = player
            value = max(value, minimax(ttt,depth-1,player%2+1,False))
            ttt[valid_move] = 0
        return value
    else:
        value = 0b1<<20
        for valid_move in get_valid_moves(grid):
            value = min(value, minimax(ttt,depth-1,player%2+1,True))
            ttt[valid_move] = 0
        return value
```


```python
def score_move(grid, depth, move, player):
    ttt = grid.copy()
    ttt[move] = player
    value = minimax(ttt, depth-1, player, False)
    return value
```


```python
def get_optimal_move(grid, depth, player):
    scores = []
    valid_moves = get_valid_moves(grid)
    for valid_move in valid_moves:
        scores.append(score_move(grid, depth, valid_move, player))
    max_score = max(scores)
    max_score_indices = [index for index,score in enumerate(scores) if score == max_score]
    return valid_moves[random.choice(max_score_indices)]
```

## Time to battle!

### Against self
```python
agent1 = {'piece':1}
agent2 = {'piece':2}

result = {'total_games':0, 'agent1':0, 'agent2':0, 'draws':0}
ngames = 100
depth = 3

for _ in range(ngames):
    grid = np.zeros((3,3), np.uint16)
    result['total_games'] += 1
    for iteration in range(9):
        # Optimal Agent
        if iteration%2 == 0:
            move = get_optimal_move(grid,depth,agent1['piece'])
            grid[move] = agent1['piece']
        # Optimal Agent
        else:
            move = get_optimal_move(grid,depth,agent2['piece'])
            grid[move] = agent2['piece']
#         print(grid)
#         sleep(2)
#         clear_output()
        if check_win_draw(grid) != -1:
            game_result = check_win_draw(grid)
            if game_result == 1:
                result['agent1'] += 1
            elif game_result == 2:
                result['agent2'] += 1
            else:
                result['draws'] += 1
            break

print_result(result)
```
```
                          Stats                       
    --------------------------------------------------
    Total Games:100   Agent1:40    Agent2:19    Draws:41   
```
### Against Random bot
```python
agent1 = {'piece':1}
agent2 = {'piece':2}

result = {'total_games':0, 'agent1':0, 'agent2':0, 'draws':0}
ngames = 100
depth = 3

for _ in range(ngames):
    grid = np.zeros((3,3), np.uint16)
    result['total_games'] += 1
    for iteration in range(9):
        if iteration%2 == 0:
            move = get_optimal_move(grid,depth,agent1['piece'])
            grid[move] = agent1['piece']
        else:
            move = get_random_tile(grid)
            grid[move] = agent2['piece']
        if check_win_draw(grid) != -1:
            game_result = check_win_draw(grid)
            if game_result == 1:
                result['agent1'] += 1
            elif game_result == 2:
                result['agent2'] += 1
            else:
                result['draws'] += 1
            break

print_result(result)
```
```
                          Stats                       
    --------------------------------------------------
    Total Games:100   Agent1:100   Agent2:0     Draws:0    
```
[article2]: {{site.baseurl}}/{% post_url 2021-11-18-one-step-lookahead-tictactoe %}
[nstepwiki]: https://en.wikipedia.org/wiki/Minimax