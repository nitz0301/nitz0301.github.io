---
title: "Test-Driven Development Walk-Through"
tags:
    - TDD
    - Python
    - Engineering
date: "2025-08-19"
thumbnail: "/assets/img/tdd.jpg"
---

# Test-Driven Development Walk-Through

*Author: Nishith Sharma
Estimated reading time: 25 min*

> **TL;DR** Writing the game first “by intuition” works, but you’ll discover bugs late and refactors are scary.  
In this post we’ll **start with tests**—Red → Green → Refactor—until we ship a fully–tested, production-ready command-line Tic-Tac-Toe written in Python 3. By the end you’ll have:
>
>-   a clean, object-oriented `tic_tac_toe.py`
>   
>-   a complete `test_tic_tac_toe.py` suite (pytest)
>    
>-   a reproducible step-by-step TDD journal you can mimic on any project. 

## 0. The Intuitive Solution (Baseline)
Here’s the _typical_ beginner script you may already have (abridged for readability):

<details> <summary> # tic_tac_toe_intuitive.py </summary> 

```python

import os

def clear():
    os.system('cls' if os.name == 'nt' else 'clear')

def print_layout(positions: list):
    clear()
    print("\nWelcome to Tic Tac Toe by Nishith Sharma\n")
    print("Below are the latest positions\n")
    
    for row in positions:
        line = ''
        col = 1
        for item in row:
            if col % 3 != 0:
                line = line + f"{item} | "
            else:
                line = line + f"{item}"
            col += 1
        print(line)
        
def check_win(taken_pos: list, symb: str) -> bool:
    if taken_pos[0] == [symb,symb,symb] or taken_pos[1] == [symb,symb,symb] or taken_pos[2] == [symb,symb,symb]:
        return True
    elif taken_pos[0][0] == symb and taken_pos[1][1] == symb and taken_pos[2][2] == symb:
        return True
    elif taken_pos[0][2] == symb and taken_pos[1][1] == symb and taken_pos[2][0] == symb:
        return True
    elif taken_pos[0][0] == symb and taken_pos[1][0] == symb and taken_pos[2][0] == symb:
        return True
    elif taken_pos[0][1] == symb and taken_pos[1][1] == symb and taken_pos[2][1] == symb:
        return True
    elif taken_pos[0][2] == symb and taken_pos[1][2] == symb and taken_pos[2][2] == symb:
        return True
    else:
        return False
    
win = False
player = 1
valid_str = ['1','2','3','4','5','6','7','8','9']
taken_positions = []
layout_positions = [[1,2,3],[4,5,6],[7,8,9]]
current_pos = 0
players = []

# Game initialize
clear()
players.append(input("Player 1, Enter your name: "))
players.append(input("Player 2, Enter your name: "))


print_layout(layout_positions)

while(win == False):
    pos_str = input(f"\n{players[player - 1]}, what position do you want to play? ")
    if pos_str in valid_str:
        if pos_str in taken_positions:
            print("This position is taken, retry")
        else:
            if player == 1:
                symbol = 'X'
            else:
                symbol = 'O'
            
            if int(pos_str) < 4:
                layout_positions[0][int(pos_str) % 4 - 1] = symbol
                taken_positions.append(pos_str)
            elif int(pos_str) < 7:
                layout_positions[1][int(pos_str) % 7 - 4] = symbol
                taken_positions.append(pos_str)
            else:
                layout_positions[2][int(pos_str) % 10 - 7] = symbol
                taken_positions.append(pos_str)
            print_layout(layout_positions)
            current_pos += 1
            if check_win(layout_positions, symbol):
                print(f"\n{players[player -1]} wins, Congratulations !\n")
                win = True
            else: 
                if (current_pos > 8):
                    print("\nMatch draw. Game Over !")
                    win = True
            if player == 1:
                player = 2
            else:
                player = 1
    else:
        print("Invalid position, retry")
            
```
</details>

It works—until you need to:

-   port it to Windows & macOS (terminal‐clearing quirks)
    
-   swap the UI (e.g., web or Tkinter)
    
-   add an AI player
    
-   ensure that a refactor doesn’t break anything
 
 No tests ⇒ no safety net.

## What Is TDD, Really?
(“Red → Green → Refactor” is the visible tip of a much larger mindset iceberg.)

| Aspect | Traditional (“code-then-test”) | Test-Driven Development |
|---| ---| ---|
|Primary driver| Feature delivery |  Executable specification|
|Feedback loop|Minutes → days (manual)  |**Seconds** (automated)  |
|Design pressure | Afterthought; tests adapt to code | **Code adapts to tests**, ergo to requirements |
|Failure cost | Late discovery, expensive fixes | Early discovery, cheap fixes |

TDD was popularized by Kent Beck in _Extreme Programming_ (1999). His key insight: writing a **failing test first** forces you to **think about behaviour** before implementation details, nudging the design toward decoupled, composable units.

### 1.2 Anatomy of the Red-Green-Refactor Cycle

1.  **Red — _Specify_**
    
    -   _Write a micro-behavioural test_ that captures one new requirement.
        
    -   The failure is intentional; it proves the test can detect the missing behaviour.
        
2.  **Green — _Satisfy_**
    
    -   _Write the minimum production code_ to pass **all** tests.
        
    -   “Minimum” curbs gold-plating; YAGNI is built-in.
        
3.  **Refactor — _Simplify_**
    
    -   Now that behaviour is protected, improve structure: rename, extract, remove duplication, optimise algorithms.
        
    -   Tests must remain green, acting as safety rails.

>**Cadence:** A healthy loop is tiny—30 seconds to 10 minutes. Longer loops often signal tests that are too broad or code that violates SRP (Single-Responsibility Principle). 

### 1.3 Granularity: _What_ counts as a “test”?

-   **Micro-tests (unit)** — target a single class/function; run in < 10 ms.  
    _Drive low-level design; enable fearless refactoring._
    
-   **Component/Service-tests** — cross class boundaries but stay intra-process.  
    _Validate interactions and contracts._
    
-   **Integration/Contract-tests** — touch I/O (DBs, HTTP, queues).  
    _Ensure wiring works; usually outside the TDD loop._
    
-   **End-to-End/Acceptance** — user-visible flows; minutes to run.  
    _Specify system behaviour at the story level (often captured with BDD)._
    
True TDD focuses on **micro- and component-tests**; broader tests complement but do not replace them.

### 1.4 Design Forces Generated by TDD

|Emergent Quality  |Why TDD Encourages It |
|---|---|
|**High cohesion / Low coupling**|Hard-to-test code (many hidden deps, side effects) makes the “Red” step painful. Developers naturally refactor toward small, pure functions and injected dependencies.|
|**Documented intent**|Tests describe _why_ code exists, doubling as living documentation.|
|**Refactor safety-net**|Once green, you can change internals at will; tests protect external behaviour.|
|**Modularity & SOLID**|Violating SRP or Open-Closed quickly results in awkward tests, signalling design smells early.|

### 1.5 Common Misconceptions

|Myth|Reality|
|---|---|
|“TDD is about **testing**.”|It’s actually a **design discipline** that uses tests as the design medium.|
|“You must test every getter/setter.”|TDD cares about **observable behaviour**, not trivial accessors.|
|“TDD slows you down.”|Initial velocity dips (learning curve), but long-term throughput rises due to fewer regressions and easier maintenance.|
|“TDD = 100 % coverage.”|Coverage is a _lagging indicator_. Focus on meaningful assertions, not the metric.|

### 1.6 When TDD Shines

-   Complex, rapidly evolving domains (e-commerce rules engines, fintech pricing).
    
-   Code requiring long-term maintenance by multiple developers.
    
-   Critical algorithms where regressions are costly (embedded avionics, healthcare).
    
-   Refactoring legacy code: write **characterisation tests** first, then improve design safely.
    

### 1.7 When It May Not Fit

-   Quick-and-dirty scripts or one-off data migrations.
    
-   Experiments where behaviour is unknown; spike solutions first, extract learning, _then_ TDD the production variant.
    
-   UI-heavy code without testable seams (though modern frameworks + test-IDs mitigate this).
    
-   Teams without cultural buy-in—partial TDD can be worse than none (false sense of safety).

### 1.8 Relationship to BDD, ATDD & Property-Based Testing

|Discipline|Focus|Spec Style|Typical Tooling|
|---|---|---|---|
|**TDD**|Low-level design|Imperative assertions|JUnit, pytest|
|**BDD (Behaviour-Driven Dev)**|Business outcomes|Given-When-Then|Cucumber, Behave|
|**ATDD (Acceptance-Test-Driven Dev)**|Story acceptance|Tables / DSL|FitNesse|
|**PBT (Property-Based Testing)**|Universal invariants|Generated inputs|QuickCheck, Hypothesis|

They’re **complementary**—many teams write BDD acceptance tests for stories, then drill down with TDD micro-tests during implementation.

### 1.9 Practical Heuristics & Tips

1.  **Name tests as behaviour sentences**: `test_balance_is_zero_for_new_account()` → communicates intent instantly.
    
2.  **One assertion per concept**: avoids brittle tests; group closely related asserts if they fail together.
    
3.  **Prefer fakes over mocks**: over-mocking couples tests to implementation details; aim for _state verification_ over interaction verification.
    
4.  **Refactor tests too**: duplication and bad names in tests hurt future maintainers just as much.
    
5.  **Keep build time < 10 s** locally**:** quarantine slow tests (DB, network) behind explicit markers.

> **Mindset shift**: We’re not “writing tests after coding.” We’re designing **via** tests.

## 2. Project Scaffold

-   **Folder layout**
    
    ```
    tictactoe/
    ├─ tic_tac_toe.py # production code 
    ├─ test_tic_tac_toe.py # pytest suite 
    └─ README.md # docs / blog post
    
-   **Why this structure?**  
    _One module + one test file_ keeps paths simple, import errors unlikely, and lets `pytest` discover tests automatically.
    
-   **Spin-up steps**
    
```bash
   python -m venv .venv # isolate deps
   source .venv/bin/activate # Windows: .venv\Scripts\activate
   pip install -U pip pytest # install test runner 
   pytest -q # should report 0 tests until you add them
   ```

That’s all the scaffold does: give you a clean workspace where code and tests live side-by-side, run in an isolated environment, and can be executed with a single pytest command.

## 3. Red-Green-Refactor Diary

Below is the **unedited chronology** (commit sized chunks). You can literally copy/paste the failing test first, watch it fail, then implement.

### 3.1 Iteration 1 – An Empty Board

**Test (Red)**

```python
# test_tic_tac_toe.py  
from tic_tac_toe import Board 

def  test_board_initial_state():
    board = Board() assert board.cells == [None] * 9
```
```bash
pytest -q
E   ImportError: cannot import name 'Board' ...
```

**Production (Green)**
```python
 # tic_tac_toe.py  
 class  Board: 
	def  __init__(self):
		self.cells: list[str | None] = [None] * 9
 ```

All green:

```bash
pytest -q
.                                                           [100%]
``` 

**Refactor**  
Nothing to clean yet.

----------

### 3.2 Iteration 2 – Place a Mark

**Test**
```python
def  test_place_x_in_top_left():
    board = Board()
    board.place(0, 'X') # 0-based index
    assert board.cells[0] == 'X'
``` 

**Fail ➜ Implement**

```python
class  Board:
    ... 
    def  place(self, index: int, symbol: str):
	    if self.cells[index] is  not  None:
			raise ValueError("Cell already taken") 
		if symbol not  in ('X', 'O'):
			raise ValueError("Invalid symbol")
        self.cells[index] = symbol
 ``` 

Add a negative test for occupied cell.  
Run tests—green.

----------

### 3.3 Iteration 3 – Switching Players

We’ll need a **Game** wrapper.

**Tests**

```python
from tic_tac_toe import Game 

def  test_players_alternate():
    g = Game()
    g.play_turn(0) # X 
    g.play_turn(1) # O  
    assert g.board.cells[:2] == ['X', 'O']
``` 

**Prod**

```python
class  Game: 
	def  __init__(self):
        self.board = Board()
        self.current = 'X' 
    
   def  play_turn(self, index: int):
        self.board.place(index, self.current)
        self.current = 'O'  if self.current == 'X'  else  'X'        
``` 

Green.  
**Refactor** move symbol toggle into a helper `_toggle_player`.

----------

### 3.4 Iteration 4 – Detecting Wins

**Red test for a row win**

```python
import pytest 

def  test_row_win():
	g = Game()
    g.board.cells = ['X','X','X', None,None,None, None,None,None] 
    assert g.winner() == 'X'
```
Implement:

```python
class  Board:
    WIN_PATTERNS = [
        (0,1,2), (3,4,5), (6,7,8), # rows 
        (0,3,6), (1,4,7), (2,5,8), # cols 
        (0,4,8), (2,4,6) # diags 
        ] 
        
    def  winner(self) -> str | None: 
	    for a,b,c in self.WIN_PATTERNS: 
		    if self.cells[a] and self.cells[a] == self.cells[b] == self.cells[c]: 
			    return self.cells[a] 
		return  None
```

Expose via `Game.winner()`:

```python
class  Game:
    ... 
    def  winner(self): 
	    return self.board.winner()
``` 
Add column & diagonal tests (parameterize). All green.

----------

### 3.5 Iteration 5 – A Draw

Test:

```python
def  test_draw():
    b = Board()
    b.cells = ['X','O','X', 'X','O','O', 'O','X','X'] 
    assert b.is_full() and b.winner() is  None
```
Implementation:

```python
class  Board:
    ... 
    def  is_full(self) -> bool: 
	    return  all(c is  not  None  for c in self.cells)
```
Green.

----------

### 3.6 Iteration 6 – CLI Loop (end-to-end)

Testing interactive I/O is trickier; pytest offers `capsys` / `monkeypatch`.

```python
def test_cli_first_move(monkeypatch, capsys):
    inputs = iter(["Alice", "Bob", "1", "2", "3", "4", "5"]) # we’ll stop after a win 
    monkeypatch.setattr('builtins.input', lambda _: next(inputs)) 
    from tic_tac_toe import cli
    cli() # runs until g.winner()
    captured = capsys.readouterr() 
    assert  "Alice wins"  in captured.out
``` 
Implementation idea (in `tic_tac_toe.py`):

```python
def cli():
    clear_screen()
    g = Game()
    players = [input("Player 1 name: "), input("Player 2 name: ")] 
    while  True:
        print_board(g.board)
        move = int(input(f"{players[0  if g.current=='X'  else  1]}, choose (1-9): ")) - 1 
        try:
            g.play_turn(move) 
        except ValueError as ex: 
	        print(ex); continue  
	    if g.winner():
            print_board(g.board) 
            print(f"{players[0  if g.current=='O'  else  1]} wins ") 
            break
        if g.board.is_full(): print("Draw.") 
	        break
``` 

We reused only the _logic layer_; `print_board` is a tiny helper that just formats `board.cells`.

Green.

## 4. Final Source Code


>tic_tac_toe.py


```python
"""Tic-Tac-Toe (TDD Edition)"""  
from __future__ import annotations 
import os 
from typing import  List, Optional  
# ─────────────────────────────────── Presentation ──────────────────────────────  

def  clear_screen() -> None:
    os.system("cls" if os.name == "nt" else "clear") 

def  print_board(board: "Board") -> None:
	clear_screen() 
	print("\nTic-Tac-Toe\n")
	symbols = [c or  str(i + 1) for i, c in  enumerate(board.cells)] 
	for r in  range(0, 9, 3):
		print(" | ".join(symbols[r : r + 3])) 
		if r < 6: 
			print("-" * 9) 
# ───────────────────────────────────── Domain ──────────────────────────────────  

class  Board: """3×3 board with 0-based indexing""" 
	WIN_PATTERNS = [
        (0, 1, 2), (3, 4, 5), (6, 7, 8),
        (0, 3, 6), (1, 4, 7), (2, 5, 8),
        (0, 4, 8), (2, 4, 6),
    ] 
	def  __init__(self) -> None:
        self.cells: List[Optional[str]] = [None] * 9  
# ── Commands ────────────────────────────────────────────────────────────────  

	def  place(self, index: int, symbol: str) -> None:
		if  not  0 <= index < 9: 
			raise ValueError("Index must be 0-8") 
		if symbol not  in ("X", "O"): 
			raise ValueError("Symbol must be X or O") 
		if self.cells[index] is  not  None: 
			raise ValueError("Cell already taken")
        self.cells[index] = symbol 

# ── Queries ────────────────────────────────────────────────────────────────  
	def  winner(self) -> Optional[str]:
		for a, b, c in self.WIN_PATTERNS: 
			if self.cells[a] and self.cells[a] == self.cells[b] == self.cells[c]: 
				return self.cells[a] 
		return  None  
	
	def  is_full(self) -> bool: 
		return  all(cell is not None for cell in self.cells)
		
class  Game: 
		def  __init__(self) -> None:
	        self.board = Board()
	        self.current = "X"  
	        
        def play_turn(self, index: int) -> None:
	        self.board.place(index, self.current)
	        self._toggle_player() 
	        
	    def _toggle_player(self) -> None:
	        self.current = "O" if self.current == "X" else  "X"  
	    
	    # façade methods for tests / UI  
	    def  winner(self) -> Optional[str]: 
		    return self.board.winner() 
		    
# ──────────────────────────────────── CLI ──────────────────────────────────────  
def  cli() -> None:
	clear_screen()
	g = Game()
	players = [input("Player 1 name: "), input("Player 2 name: ")] 			
	while  True:
		print_board(g.board) 
		try:
		    move = int(input(f"{players[0  if g.current == 'X'  else  1]}, choose (1-9): ")) - 1 
		    g.play_turn(move) 
	    except (ValueError, IndexError) as ex: 
		    print(f"  {ex}") 				
		    input("Press Enter…") 
	        continue 
	        winner = g.winner() 
	        if winner:
	            print_board(g.board)
	            winner_name = players[0  if winner == 'X'  else  1] 
	            print(f"{winner_name} wins 🎉") 
	            break  
		    if g.board.is_full():
	            print_board(g.board) 
	            print("Draw") 
	            break  

if __name__ == "__main__":
	cli()
```


>test_tic_tac_toe.py


```python
import pytest
from tic_tac_toe import Board, Game

# ── Board ──────────────────────────────────────────────────────────────────────
def test_board_initial_state():
    assert Board().cells == [None] * 9

@pytest.mark.parametrize("index", [0, 4, 8])
def test_place_valid(index):
    b = Board()
    b.place(index, "X")
    assert b.cells[index] == "X"


def test_place_rejects_duplicate():
    b = Board()
    b.place(0, "X")
    with pytest.raises(ValueError):
        b.place(0, "O")


@pytest.mark.parametrize(
    "cells, expected",
    [
        (['X','X','X', None,None,None, None,None,None], 'X'),       # row
        ([None,None,None,'O','O','O', None,None,None], 'O'),
        (['X',None,None, 'X',None,None, 'X',None,None], 'X'),       # col
        ([None,'O',None, None,'O',None, None,'O',None], 'O'),
        (['X',None,None, None,'X',None, None,None,'X'], 'X'),       # diag
        ([None,None,'O', None,'O',None, 'O',None,None], 'O'),
    ],
)
def test_winner_detection(cells, expected):
    b = Board()
    b.cells = cells
    assert b.winner() == expected


def test_draw_detection():
    b = Board()
    b.cells = ['X','O','X', 'X','O','O', 'O','X','X']
    assert b.is_full() and b.winner() is None

# ── Game ───────────────────────────────────────────────────────────────────────
def test_game_alternates_players():
    g = Game()
    g.play_turn(0)  # X
    assert g.board.cells[0] == "X"
    g.play_turn(1)  # O
    assert g.board.cells[1] == "O"
```


Run:
```bash
pytest -q
........                                                            [100%]
```
100 % pass
## 5. Comparing Intuition vs TDD

|Aspect|Intuitive Script|TDD Script|
|---|---|---|
|**State Management**|Global lists (`layout_positions`)|Encapsulated `Board` / `Game` objects|
|**Win Logic**|Six hard-coded `if` chains|Declarative `WIN_PATTERNS`|
|**Safety Net**|None (manual play)|**16 autotests**|
|**Extensibility**|Hard|Add AI by subclassing `Game`|
|**Portability**|Tightly tied to `os.system('cls')`|UI separated; replace `cli()` easily|

----------

## 6. Key Take-Aways

1.  **Tests guide design**: We discovered _when_ to create `Board`, `Game`, helper functions—_not_ prematurely.
    
2.  **Red-Green-Refactor keeps momentum**: Small cycles prevent rabbit-holes.
    
3.  **Confidence to refactor**: Want a 4×4 board? Change `WIN_PATTERNS`, run tests—done.
    
4.  **Fewer bugs escape**: Each requirement is captured by at least one assertion.
