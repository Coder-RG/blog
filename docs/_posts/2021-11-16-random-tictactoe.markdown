---
layout: post
title: "Bot plays Tic-Tac-Toe using Random moves"
author: Coder-RG
categories: reinforcement-learning
---

Welcome! This is the first article in the reinforcement learning path. In this path we will develop
an agent that will be able to beat the best. Got too excited there, moving on!
The agent will progress in stages. In each blog, it will become smarter in how it chooses its moves. Let's get started, shall we?

### Introduction to Tic-Tac-Toe
If you didn't undertand what I talked about till now, let me give a refresher. Tic-Tac-Toe consists
of a 3x3 grid and two players compete to win. One player is assigned the marker **X** and the other is assigned the marker **O**. The play alternates between the two players i.e. each player can only play in their turn. The first player to place their marker in a sequence wins. This sequence can be single row, column or diagonal.

{:refdef: style="text-align: center;"}
![Tic-Tac-Toe]({{site.baseurl}}/assets/images/tictactoe.png)
{: refdef}

### Import the necessary packages

```python
import numpy as np
import random
from IPython.display import clear_output
from time import sleep
```

### Create the battlefield

Before creating any agent, it is necessary to create the grid on which the game will be played.

```python
ttt = np.zeros((3,3), np.uint16)
ttt
```
```
array([[0, 0, 0],
       [0, 0, 0],
       [0, 0, 0]], dtype=uint16)
```

### Time for some rules

After every play, we need to check if any player has won. The code below will help us in this task.

```python
def check_win(ttt):
    """
    Checks for the win condition i.e, 3 sequential markers of one player.

    Paramters:
    ==========
    ttt: np.ndarray
        --> 3x3 Tic-Tac-Toe grid
    
    Returns:
    ========
    : int
        --> 0,1,2 where each value represent who won. 0 if none, 1 for player 1
            and 2 for player 2.
    """
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
    return 0

```
Everything has been setup, in context to the environment. Next, we will create the code for the agent's moves. Since our agent has only just started with this game, it will be a bit dumb. How dumb? It will play random moves.

```python
def get_random_tile(ttt):
    """
    Get a random location for the grid where play is possible. Only tile value
    '0' represent a valid move.

    Parameters:
    ===========
    ttt: np.ndarray
        --> 3x3 Tic-Tac-Toe grid.

    Returns:
    ========
    (row,col): tuple
        --> A tuple with tile location on the grid. Remember the grid is 0
            indexed.
    """
    while True:
        row = random.randint(0,2)
        col = random.randint(0,2)
        if ttt[row,col] == 0:
            return (row,col)

```

This is where things get exciting. All the necessary code has been developed. The agent can safely play some games now.

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
        # Agents play alternatively
        move = get_random_tile(ttt)
        if iteration%2 == 0:
            ttt[move] = agent1['piece']
        else:
            ttt[move] = agent2['piece']
        winner = check_win(ttt)
        if winner != 0:
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
Total Games:100   Agent1:59    Agent2:28    Draws:13   
```

Our agent performs really poorly, which should be expected. Playing random moves is not guaranteed to win games. However, our agent still wins 59% of the games.

This article has just laid the groundworks. Complete code is not shown above, since it was not relevant. You can check the complete code in jupyter notebook [here](https://github.com/Coder-RG/TTTWithDumbAgent/blob/master/TTTWithDumbAgent.ipynb)