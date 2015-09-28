
# CS 182: Artificial Intelligence
# Assignment 3: Constraint Satisfaction and Local Search
* Fall 2015
* Due: Oct 16, 5pm


## Solving Sudoku as a Constraint Satisfaction Problem (CSP)

In a traditional search problem (as in HW1), we treat each state of a problem abstractly. A state has a goal test and a successor function, but we never look inside the representation of a state. In a constraint satisfaction problem (CSP), however, a state is no longer opaque. We can peek into the state to check if we are on the right track. In this problem set, you will implement a Sudoku solver that leverages a few common techniques for solving CSPs. The code for this project is in `sudoku.py`.


```python
from sudoku import *
import IPython.display
```

### Problem 0: Introduction to Sudoku
If you are not familiar with Sudoku puzzles, read the Wikipedia page on Sudoku to familiarize yourself with the puzzle.

https://en.wikipedia.org/wiki/Sudoku

Here is an example of a Sudoku problem:


```python
sudoku = Sudoku(boardHard)
IPython.display.HTML(sudoku.printhtml())
```




<style>
         .sudoku td {
            width: 20pt;
            text-align: center;
            border-color: #AAAAAA;
         }

        </style><center><table class='sudoku' style='border:none; border-collapse:collapse; background-color:#FFFFFF; border: #666699 solid 2px;'><tr style='border: #AAAAAA 1px'><td style=''> </td><td style=''> </td><td style=''> </td><td style='border-left: #666699 solid 2px;'> </td><td style=''> </td><td style=''>8</td><td style='border-left: #666699 solid 2px;'>9</td><td style=''> </td><td style=''>2</td></tr><tr style='border: #AAAAAA 1px'><td style=''>6</td><td style=''> </td><td style=''>4</td><td style='border-left: #666699 solid 2px;'>3</td><td style=''> </td><td style=''> </td><td style='border-left: #666699 solid 2px;'> </td><td style=''> </td><td style=''> </td></tr><tr style='border: #AAAAAA 1px'><td style=''> </td><td style=''> </td><td style=''> </td><td style='border-left: #666699 solid 2px;'>5</td><td style=''>9</td><td style=''> </td><td style='border-left: #666699 solid 2px;'> </td><td style=''> </td><td style=''> </td></tr><tr style='border:none; border-collapse:collapse; background-color:#AAAAAA 1px; border-top: #666699 solid 2px'><td style=''> </td><td style=''> </td><td style=''>5</td><td style='border-left: #666699 solid 2px;'>7</td><td style=''> </td><td style=''>3</td><td style='border-left: #666699 solid 2px;'> </td><td style=''> </td><td style=''>9</td></tr><tr style='border: #AAAAAA 1px'><td style=''>7</td><td style=''> </td><td style=''> </td><td style='border-left: #666699 solid 2px;'> </td><td style=''>4</td><td style=''> </td><td style='border-left: #666699 solid 2px;'> </td><td style=''> </td><td style=''> </td></tr><tr style='border: #AAAAAA 1px'><td style=''> </td><td style=''> </td><td style=''>9</td><td style='border-left: #666699 solid 2px;'> </td><td style=''> </td><td style=''> </td><td style='border-left: #666699 solid 2px;'>3</td><td style=''> </td><td style=''>5</td></tr><tr style='border:none; border-collapse:collapse; background-color:#AAAAAA 1px; border-top: #666699 solid 2px'><td style=''> </td><td style=''>8</td><td style=''> </td><td style='border-left: #666699 solid 2px;'> </td><td style=''> </td><td style=''>4</td><td style='border-left: #666699 solid 2px;'> </td><td style=''> </td><td style=''> </td></tr><tr style='border: #AAAAAA 1px'><td style=''> </td><td style=''>4</td><td style=''>1</td><td style='border-left: #666699 solid 2px;'> </td><td style=''> </td><td style=''> </td><td style='border-left: #666699 solid 2px;'> </td><td style=''>3</td><td style=''> </td></tr><tr style='border: #AAAAAA 1px'><td style=''>2</td><td style=''> </td><td style=''> </td><td style='border-left: #666699 solid 2px;'>1</td><td style=''>5</td><td style=''> </td><td style='border-left: #666699 solid 2px;'> </td><td style=''> </td><td style=''> </td></tr></table></center>



We also implement a method `prettyprint` which will show the true state of the CSP problem. Each column shows the labels that still can be used in each column and row, as well as the number of violations (note we do not show the remaining values in the boxes). Since the beginning state is consistent, these are all zero. Also each unassigned variable is shown with the number of remaining possible assignments. 


```python
IPython.display.HTML(sudoku.prettyprinthtml())
```




<style>
         .sudoku .board {
            width: 20pt;
            text-align: center;
            border-color: #AAAAAA;
         }

         .sudoku .out {
            width: 10pt;
            text-align: center;
            border-color: #FFFFFF;
         }

         .sudoku .outtop {
            padding: 0pt;
            text-align: center;
            border-color: #FFFFFF;
         }

        </style><center><table class='sudoku' style='border:none;border-collapse:collapse;  background-color:#FFFFFF; border: #666699 solid 2px;'><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'> 1 </td><td class='outtop' style=''> 1 </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''> 1 </td><td class='outtop' style=''> 1 </td><td class='outtop' style='border-left: #666699 solid 2px;'> 1 </td><td class='outtop' style=''> 1 </td><td class='outtop' style='border-right: #666699 solid 2px;'> 1 </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''> 2 </td><td class='outtop' style=''> 2 </td><td class='outtop' style='border-left: #666699 solid 2px;'> 2 </td><td class='outtop' style=''> 2 </td><td class='outtop' style=''> 2 </td><td class='outtop' style='border-left: #666699 solid 2px;'> 2 </td><td class='outtop' style=''> 2 </td><td class='outtop' style='border-right: #666699 solid 2px;'>     </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'> 3 </td><td class='outtop' style=''> 3 </td><td class='outtop' style=''> 3 </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''> 3 </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''>     </td><td class='outtop' style='border-right: #666699 solid 2px;'> 3 </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'> 4 </td><td class='outtop' style=''>     </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'> 4 </td><td class='outtop' style=''>     </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'> 4 </td><td class='outtop' style=''> 4 </td><td class='outtop' style='border-right: #666699 solid 2px;'> 4 </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'> 5 </td><td class='outtop' style=''> 5 </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''>     </td><td class='outtop' style=''> 5 </td><td class='outtop' style='border-left: #666699 solid 2px;'> 5 </td><td class='outtop' style=''> 5 </td><td class='outtop' style='border-right: #666699 solid 2px;'>     </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''> 6 </td><td class='outtop' style=''> 6 </td><td class='outtop' style='border-left: #666699 solid 2px;'> 6 </td><td class='outtop' style=''> 6 </td><td class='outtop' style=''> 6 </td><td class='outtop' style='border-left: #666699 solid 2px;'> 6 </td><td class='outtop' style=''> 6 </td><td class='outtop' style='border-right: #666699 solid 2px;'> 6 </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''> 7 </td><td class='outtop' style=''> 7 </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''> 7 </td><td class='outtop' style=''> 7 </td><td class='outtop' style='border-left: #666699 solid 2px;'> 7 </td><td class='outtop' style=''> 7 </td><td class='outtop' style='border-right: #666699 solid 2px;'> 7 </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'> 8 </td><td class='outtop' style=''>     </td><td class='outtop' style=''> 8 </td><td class='outtop' style='border-left: #666699 solid 2px;'> 8 </td><td class='outtop' style=''> 8 </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'> 8 </td><td class='outtop' style=''> 8 </td><td class='outtop' style='border-right: #666699 solid 2px;'> 8 </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'> 9 </td><td class='outtop' style=''> 9 </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'> 9 </td><td class='outtop' style=''>     </td><td class='outtop' style=''> 9 </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''> 9 </td><td class='outtop' style='border-right: #666699 solid 2px;'>     </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border:none; border-collapse:collapse; background-color:#AAAAAA 1px; border-top: #666699 solid 2px'><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''> </td><td class='board' style=''> </td><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''> </td><td class='board' style=''>8</td><td class='board' style='border-left: #666699 solid 2px;'>9</td><td class='board' style=''> </td><td class='board' style='border-right: #666699 solid 2px;'>2</td><td class='out'>1</td><td class='out'> </td><td class='out'>3</td><td class='out'>4</td><td class='out'>5</td><td class='out'>6</td><td class='out'>7</td><td class='out'> </td><td class='out'> </td></tr><tr style='border: #AAAAAA 1px'><td class='board' style='border-left: #666699 solid 2px;'>6</td><td class='board' style=''> </td><td class='board' style=''>4</td><td class='board' style='border-left: #666699 solid 2px;'>3</td><td class='board' style=''> </td><td class='board' style=''> </td><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''> </td><td class='board' style='border-right: #666699 solid 2px;'> </td><td class='out'>1</td><td class='out'>2</td><td class='out'> </td><td class='out'> </td><td class='out'>5</td><td class='out'> </td><td class='out'>7</td><td class='out'>8</td><td class='out'>9</td></tr><tr style='border: #AAAAAA 1px'><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''> </td><td class='board' style=''> </td><td class='board' style='border-left: #666699 solid 2px;'>5</td><td class='board' style=''>9</td><td class='board' style=''> </td><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''> </td><td class='board' style='border-right: #666699 solid 2px;'> </td><td class='out'>1</td><td class='out'>2</td><td class='out'>3</td><td class='out'>4</td><td class='out'> </td><td class='out'>6</td><td class='out'>7</td><td class='out'>8</td><td class='out'> </td></tr><tr style='border:none; border-collapse:collapse; background-color:#AAAAAA 1px; border-top: #666699 solid 2px'><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''> </td><td class='board' style=''>5</td><td class='board' style='border-left: #666699 solid 2px;'>7</td><td class='board' style=''> </td><td class='board' style=''>3</td><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''> </td><td class='board' style='border-right: #666699 solid 2px;'>9</td><td class='out'>1</td><td class='out'>2</td><td class='out'> </td><td class='out'>4</td><td class='out'> </td><td class='out'>6</td><td class='out'> </td><td class='out'>8</td><td class='out'> </td></tr><tr style='border: #AAAAAA 1px'><td class='board' style='border-left: #666699 solid 2px;'>7</td><td class='board' style=''> </td><td class='board' style=''> </td><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''>4</td><td class='board' style=''> </td><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''> </td><td class='board' style='border-right: #666699 solid 2px;'> </td><td class='out'>1</td><td class='out'>2</td><td class='out'>3</td><td class='out'> </td><td class='out'>5</td><td class='out'>6</td><td class='out'> </td><td class='out'>8</td><td class='out'>9</td></tr><tr style='border: #AAAAAA 1px'><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''> </td><td class='board' style=''>9</td><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''> </td><td class='board' style=''> </td><td class='board' style='border-left: #666699 solid 2px;'>3</td><td class='board' style=''> </td><td class='board' style='border-right: #666699 solid 2px;'>5</td><td class='out'>1</td><td class='out'>2</td><td class='out'> </td><td class='out'>4</td><td class='out'> </td><td class='out'>6</td><td class='out'>7</td><td class='out'>8</td><td class='out'> </td></tr><tr style='border:none; border-collapse:collapse; background-color:#AAAAAA 1px; border-top: #666699 solid 2px'><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''>8</td><td class='board' style=''> </td><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''> </td><td class='board' style=''>4</td><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''> </td><td class='board' style='border-right: #666699 solid 2px;'> </td><td class='out'>1</td><td class='out'>2</td><td class='out'>3</td><td class='out'> </td><td class='out'>5</td><td class='out'>6</td><td class='out'>7</td><td class='out'> </td><td class='out'>9</td></tr><tr style='border: #AAAAAA 1px'><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''>4</td><td class='board' style=''>1</td><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''> </td><td class='board' style=''> </td><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''>3</td><td class='board' style='border-right: #666699 solid 2px;'> </td><td class='out'> </td><td class='out'>2</td><td class='out'> </td><td class='out'> </td><td class='out'>5</td><td class='out'>6</td><td class='out'>7</td><td class='out'>8</td><td class='out'>9</td></tr><tr style='border: #AAAAAA 1px'><td class='board' style='border-left: #666699 solid 2px;'>2</td><td class='board' style=''> </td><td class='board' style=''> </td><td class='board' style='border-left: #666699 solid 2px;'>1</td><td class='board' style=''>5</td><td class='board' style=''> </td><td class='board' style='border-left: #666699 solid 2px;'> </td><td class='board' style=''> </td><td class='board' style='border-right: #666699 solid 2px;'> </td><td class='out'> </td><td class='out'> </td><td class='out'>3</td><td class='out'>4</td><td class='out'> </td><td class='out'>6</td><td class='out'>7</td><td class='out'>8</td><td class='out'>9</td></tr></table></center>



You will want to be familar with the Sudoku board and interface for accessing rows, columns, and boxes. 


```python
sudoku.board
```




    [[0, 0, 0, 0, 0, 8, 9, 0, 2],
     [6, 0, 4, 3, 0, 0, 0, 0, 0],
     [0, 0, 0, 5, 9, 0, 0, 0, 0],
     [0, 0, 5, 7, 0, 3, 0, 0, 9],
     [7, 0, 0, 0, 4, 0, 0, 0, 0],
     [0, 0, 9, 0, 0, 0, 3, 0, 5],
     [0, 8, 0, 0, 0, 4, 0, 0, 0],
     [0, 4, 1, 0, 0, 0, 0, 3, 0],
     [2, 0, 0, 1, 5, 0, 0, 0, 0]]




```python
print sudoku.row(1)
print sudoku.col(3)
print sudoku.box(0)
```

    [6, 0, 4, 3, 0, 0, 0, 0, 0]
    [0, 3, 5, 7, 0, 0, 0, 0, 1]
    [0, 0, 0, 6, 0, 4, 0, 0, 0]


### Problem 1: CSP Variables

For this assignment we will be modelling Sudoku as a CSP. Each box in the grid represents a variable based on `(row, col)`. The variable is either assigned a value or $\epsilon$ when it has not yet been assigned. Given the current assignment, the *domain* of each variable is also limited. When all the variables have been assigned the assignment is complete. For the first problem you should implement the following functions in the `Sudoku` class which model the variables of the sudoku CSP. 


```python
doc(sudoku.firstEpsilonVariable)
doc(sudoku.complete) 
doc(sudoku.variableDomain) 
```

    <bound method Sudoku.firstEpsilonVariable of <sudoku.Sudoku instance at 0x7f604c0c0758>>
    
            IMPLEMENT FOR PART 1
            Returns the first variable with assignment epsilon
            i.e. first square in the board that is unassigned.
            
    <bound method Sudoku.complete of <sudoku.Sudoku instance at 0x7f604c0c0758>>
    
            IMPLEMENT FOR PART 1
            Returns true if the assignment is complete. 
            
    <bound method Sudoku.variableDomain of <sudoku.Sudoku instance at 0x7f604c0c0758>>
    
            IMPLEMENT FOR PART 1
            Returns current domain for the (row, col) variable .
            


### Problem 2: CSP Factors

Next we will implement the factors of the CSP. The rules of Sudoku say that there must be labels from 1-9 in each row, column, and box. Each of these will be represented by factors `(type, id)`, for instance `(ROW, 2)` is the factor corresponding to the third-row. For this problem, you should implement functions which keep track of the remaining labels available for a given factor as well as the number of violation of that factor in the case of inconsistent assignments. To do this, you should implement the following functions:


```python
doc(sudoku.updateFactor)
doc(sudoku.updateAllFactors)
doc(sudoku.updateVariableFactors)
```

    <bound method Sudoku.updateFactor of <sudoku.Sudoku instance at 0x7f604c0c0758>>
    
            IMPLEMENT FOR PART 2
            Update the values remaining for a factor.
            `factor_type` is one of BOX, ROW, COL 
            `i` is an index between 0 and 8.
            
    <bound method Sudoku.updateAllFactors of <sudoku.Sudoku instance at 0x7f604c0c0758>>
    
            IMPLEMENT FOR PART 2
            Update the values remaining for all factors.
            There is one factor for each row, column, and box.
            
    <bound method Sudoku.updateVariableFactors of <sudoku.Sudoku instance at 0x7f604c0c0758>>
    
            IMPLEMENT FOR PART 2
            Update all the factors impacting a variable (neighbors in factor graph).
            


### Problem 3: Solving Sudoku with backtracking

The `solveCSP` function will simply perform a depth first search on a tree of generic problem states. Running this function requires getting the next variable to search and all the possible labels that variable can take on. First you should implement the function `getSuccessors` which returns all possible successor assignments resulting from assigning a label to a variable. This function should will need to call `setVariable`


```python
doc(sudoku.getSuccessors)
```

    <bound method Sudoku.getSuccessors of <sudoku.Sudoku instance at 0x7f604c0c0758>>
    
            IMPLEMENT IN PART 3
            Returns new assignments with each possible value 
            assigned to the variable returned by `nextVariable`.
            


After you have implemented this method, run the program by running.


```python
#!python pset3.py 
# or to see each stage use
#!python pset3.py --debug=1
```

Take note of how many states the CSP solver explores, and how much time it takes to solve the puzzle.

### Problem 4: Improving performance with Forward Checking.

Now, you will try to improve the performance of your Sudoku solver by implementing forward checking. First, take a look at: the function `getSuccessorsWithForwardChecking()` and then implement `forwardCheck`



```python
doc(sudoku.forwardCheck)
```

    <bound method Sudoku.forwardCheck of <sudoku.Sudoku instance at 0x7f604c0c0758>>
    
            IMPLEMENT IN PART 4
            Returns trues if all variables have non-empty domains.
            


Run the program and take note of the number of states explored and how much time it takes.



```python
#!python pset3.py --forward 1
```

### Problem 5: Improving performance with ordering
In CSPs, we can often vastly improve algorithms by intelligently choosing the ordering of variables to which to assign values and the ordering of the values assigned to variables. In this assignment, you will implement the minimum remaining values (MRV) heuristic to determine the ordering of variables.

Implement `mostConstrainedVariable()`.


```python
doc(sudoku.mostConstrainedVariable)
```

    <bound method Sudoku.mostConstrainedVariable of <sudoku.Sudoku instance at 0x7f604c0c0758>>
    
            IMPLEMENT IN PART 5
            Returns the most constrained unassigned variable.
            


Try running the algorithm again, both with and without forward checking. Take note of the number of nodes explored and the time elapsed.


```python
#!python pset3.py --forward 1 --mostconstrained 1
```

### Problem 5: Analyzing the performance of your solvers (submitted individually)
In a separate document, compare your results from Part 3, 4, and 5. Your analysis should report the number of nodes expanded by each of the Sudoku solvers and the time it took the solvers to find a solution. Did your algorithm improve? Why or why not? Explain in 2-4 sentences.

Analyze the performance of the solver using the minimum remaining heuristic, with and without forward checking. Your analysis should report the number of nodes expanded by each of the Sudoku solvers and the time it took the solvers to find a solution.  

Did ordering improve your performance? Does forward checking improve the algorithm when used in combination with ordering? Why or why not? Discuss your results.

### Problem 6: Local Search: Random Restarts

In the next several problems we consider a different approach to finding a solution to Sudoku, local search. In 
local search instead of working with consistent, incomplete assignments, we will instead use inconsistent, complete assignments. To start we need to sample a random complete assignment. 

In class, we started with a fully random assignment. However, it is often better to start with some randomness but also satisfying some of the factors. In particular we will sample from assignments with all of the row factors satisfied. 

You should implement the function `randomRestart`:


```python
doc(sudoku.randomRestart)
```

    <bound method Sudoku.randomRestart of <sudoku.Sudoku instance at 0x7f604c0c0758>>
    
            IMPLEMENT FOR PART 6
            Randomly initialize a complete, inconsistent board
            with all the row factors being held consistent. 
            


When this works you should be able to call `prettyprint` and see zeros along the rows but not the columns, for example


```python
board = [[4, 2, 5, 1, 7, 8, 9, 3, 6], [7, 4, 6, 3, 5, 2, 1, 9, 8], [1, 2, 8, 5, 7, 4, 9, 3, 6], [2, 1, 8, 6, 9, 3, 5, 4, 7], [3, 1, 2, 6, 9, 8, 5, 7, 4 
], [1, 5, 6, 7, 4, 9, 2, 3, 8], [9, 4, 5, 3, 8, 1, 6, 7, 2], [3, 8, 2, 9, 7, 1, 5, 6, 4], [9, 1, 8, 4, 3, 6, 7, 5, 2]]
IPython.display.HTML(Sudoku(board).prettyprinthtml())
```




<style>
         .sudoku .board {
            width: 20pt;
            text-align: center;
            border-color: #AAAAAA;
         }

         .sudoku .out {
            width: 10pt;
            text-align: center;
            border-color: #FFFFFF;
         }

         .sudoku .outtop {
            padding: 0pt;
            text-align: center;
            border-color: #FFFFFF;
         }

        </style><center><table class='sudoku' style='border:none;border-collapse:collapse;  background-color:#FFFFFF; border: #666699 solid 2px;'><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''>     </td><td class='outtop' style=''> 1 </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''> 1 </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''> 1 </td><td class='outtop' style='border-right: #666699 solid 2px;'> 1 </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''>     </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'> 2 </td><td class='outtop' style=''> 2 </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''> 2 </td><td class='outtop' style='border-right: #666699 solid 2px;'>     </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''> 3 </td><td class='outtop' style=''> 3 </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''>     </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'> 3 </td><td class='outtop' style=''>     </td><td class='outtop' style='border-right: #666699 solid 2px;'> 3 </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''>     </td><td class='outtop' style=''> 4 </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''>     </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'> 4 </td><td class='outtop' style=''>     </td><td class='outtop' style='border-right: #666699 solid 2px;'>     </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'> 5 </td><td class='outtop' style=''>     </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''>     </td><td class='outtop' style=''> 5 </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''>     </td><td class='outtop' style='border-right: #666699 solid 2px;'> 5 </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'> 6 </td><td class='outtop' style=''> 6 </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''> 6 </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''>     </td><td class='outtop' style='border-right: #666699 solid 2px;'>     </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''> 7 </td><td class='outtop' style=''> 7 </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''>     </td><td class='outtop' style=''> 7 </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''>     </td><td class='outtop' style='border-right: #666699 solid 2px;'>     </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'> 8 </td><td class='outtop' style=''>     </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'> 8 </td><td class='outtop' style=''>     </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'> 8 </td><td class='outtop' style=''> 8 </td><td class='outtop' style='border-right: #666699 solid 2px;'>     </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border: none;'><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''> 9 </td><td class='outtop' style=''> 9 </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''>     </td><td class='outtop' style=''>     </td><td class='outtop' style='border-left: #666699 solid 2px;'>     </td><td class='outtop' style=''>     </td><td class='outtop' style='border-right: #666699 solid 2px;'> 9 </td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td><td class='outtop'></td></tr><tr style='border:none; border-collapse:collapse; background-color:#AAAAAA 1px; border-top: #666699 solid 2px'><td class='board' style='border-left: #666699 solid 2px;'>4</td><td class='board' style=''>2</td><td class='board' style=''>5</td><td class='board' style='border-left: #666699 solid 2px;'>1</td><td class='board' style=''>7</td><td class='board' style=''>8</td><td class='board' style='border-left: #666699 solid 2px;'>9</td><td class='board' style=''>3</td><td class='board' style='border-right: #666699 solid 2px;'>6</td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td></tr><tr style='border: #AAAAAA 1px'><td class='board' style='border-left: #666699 solid 2px;'>7</td><td class='board' style=''>4</td><td class='board' style=''>6</td><td class='board' style='border-left: #666699 solid 2px;'>3</td><td class='board' style=''>5</td><td class='board' style=''>2</td><td class='board' style='border-left: #666699 solid 2px;'>1</td><td class='board' style=''>9</td><td class='board' style='border-right: #666699 solid 2px;'>8</td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td></tr><tr style='border: #AAAAAA 1px'><td class='board' style='border-left: #666699 solid 2px;'>1</td><td class='board' style=''>2</td><td class='board' style=''>8</td><td class='board' style='border-left: #666699 solid 2px;'>5</td><td class='board' style=''>7</td><td class='board' style=''>4</td><td class='board' style='border-left: #666699 solid 2px;'>9</td><td class='board' style=''>3</td><td class='board' style='border-right: #666699 solid 2px;'>6</td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td></tr><tr style='border:none; border-collapse:collapse; background-color:#AAAAAA 1px; border-top: #666699 solid 2px'><td class='board' style='border-left: #666699 solid 2px;'>2</td><td class='board' style=''>1</td><td class='board' style=''>8</td><td class='board' style='border-left: #666699 solid 2px;'>6</td><td class='board' style=''>9</td><td class='board' style=''>3</td><td class='board' style='border-left: #666699 solid 2px;'>5</td><td class='board' style=''>4</td><td class='board' style='border-right: #666699 solid 2px;'>7</td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td></tr><tr style='border: #AAAAAA 1px'><td class='board' style='border-left: #666699 solid 2px;'>3</td><td class='board' style=''>1</td><td class='board' style=''>2</td><td class='board' style='border-left: #666699 solid 2px;'>6</td><td class='board' style=''>9</td><td class='board' style=''>8</td><td class='board' style='border-left: #666699 solid 2px;'>5</td><td class='board' style=''>7</td><td class='board' style='border-right: #666699 solid 2px;'>4</td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td></tr><tr style='border: #AAAAAA 1px'><td class='board' style='border-left: #666699 solid 2px;'>1</td><td class='board' style=''>5</td><td class='board' style=''>6</td><td class='board' style='border-left: #666699 solid 2px;'>7</td><td class='board' style=''>4</td><td class='board' style=''>9</td><td class='board' style='border-left: #666699 solid 2px;'>2</td><td class='board' style=''>3</td><td class='board' style='border-right: #666699 solid 2px;'>8</td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td></tr><tr style='border:none; border-collapse:collapse; background-color:#AAAAAA 1px; border-top: #666699 solid 2px'><td class='board' style='border-left: #666699 solid 2px;'>9</td><td class='board' style=''>4</td><td class='board' style=''>5</td><td class='board' style='border-left: #666699 solid 2px;'>3</td><td class='board' style=''>8</td><td class='board' style=''>1</td><td class='board' style='border-left: #666699 solid 2px;'>6</td><td class='board' style=''>7</td><td class='board' style='border-right: #666699 solid 2px;'>2</td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td></tr><tr style='border: #AAAAAA 1px'><td class='board' style='border-left: #666699 solid 2px;'>3</td><td class='board' style=''>8</td><td class='board' style=''>2</td><td class='board' style='border-left: #666699 solid 2px;'>9</td><td class='board' style=''>7</td><td class='board' style=''>1</td><td class='board' style='border-left: #666699 solid 2px;'>5</td><td class='board' style=''>6</td><td class='board' style='border-right: #666699 solid 2px;'>4</td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td></tr><tr style='border: #AAAAAA 1px'><td class='board' style='border-left: #666699 solid 2px;'>9</td><td class='board' style=''>1</td><td class='board' style=''>8</td><td class='board' style='border-left: #666699 solid 2px;'>4</td><td class='board' style=''>3</td><td class='board' style=''>6</td><td class='board' style='border-left: #666699 solid 2px;'>7</td><td class='board' style=''>5</td><td class='board' style='border-right: #666699 solid 2px;'>2</td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td><td class='out'> </td></tr></table></center>



### Problem 7: Local Search: Neighbors

Local search algorithms also require being able to produce neighbors for a given assignment. One way to produce neighbors 
is to change variables at random. 

For Sudoku we can be a bit more clever and maintain consistency along the row factors. To do this we randomly select a row and swap two of the entries, being careful not to change any of the original values. For this problem you will implement this function as `randomSwap`


```python
doc(sudoku.randomSwap)
```

    <bound method Sudoku.randomSwap of <sudoku.Sudoku instance at 0x7f604c0c0758>>
    
            IMPLEMENT FOR PART 7
            Returns two random variables that can be swapped without
            causing a row factor conflict.
            


### Problem 8: Local Search: Discrete Stocastic Gradient Descent

Finally, you will implement the full local search algorithm. The algorithm should start with a random assignment with 
consistent rows. Each iteration should sample a neighbor, it has a better score `f` then we move to that neighbor, otherwise we return to the original state.  


For the scoring function `f` we use the current number of constraints that are violated. For efficiency, our implementation keeps a running count of the violations of each assignment. You will have to understand this representation and implement the following function.


```python
doc(sudoku.gradientDescent)
```

    <bound method Sudoku.gradientDescent of <sudoku.Sudoku instance at 0x7f604c0c0758>>
    
            IMPLEMENT FOR PART 7
            Decide if we should swap the values of variable1 and variable2.
            


To run the algorithm, we use the following commandline. Note that for this problem we will use the easier sudoku board as local search is less effective than standard DFS.


```python
#!python pset3.py --easy 1 --localsearch=1
```


## Submission instructions

1. Submit the written questions as a single PDF document to the dropbox.

2. For the computational part, submit only the file `sudoku.py`. If you work with a partner, only one of you should submit this file. The names of both students should appear in the file name (e.g., PeterBang_MingYin_assignment3) and at the top of the file as a comment. Make sure to document your code!

3. Please post questions on Piazza. You are encouraged to answer others’ questions. 

4. Email the TFs if you are taking a late day. 
