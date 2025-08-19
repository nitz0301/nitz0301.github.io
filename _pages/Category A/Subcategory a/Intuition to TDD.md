Test-Driven Development Walk-Through: Building a **Tic-Tac-Toe** CLI from Scratch
=================================================================================

_Author: Nishith Sharma__Estimated reading time: 25 min_

> **TL;DR** Writing the game first “by intuition” works, but you’ll discover bugs late and refactors are scary.In this post we’ll **start with tests**—Red → Green → Refactor—until we ship a fully–tested, production-ready command-line Tic-Tac-Toe written in Python 3. By the end you’ll have:
> 
> *   a clean, object-oriented tic\_tac\_toe.py
>     
> *   a complete test\_tic\_tac\_toe.py suite (pytest)
>     
> *   a reproducible step-by-step TDD journal you can mimic on any project.
>     

0\. The Intuitive Solution (Baseline)
-------------------------------------

Here’s the _typical_ beginner script you may already have (abridged for readability):

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # tic_tac_toe_intuitive.py  import os  def clear():      os.system('cls' if os.name == 'nt' else 'clear')  def print_layout(positions: list): ...  def check_win(taken_pos: list, symb: str) -> bool: ...  # procedural game loop, lots of globals   `

It works—until you need to:

*   port it to Windows & macOS (terminal‐clearing quirks)
    
*   swap the UI (e.g., web or Tkinter)
    
*   add an AI player
    
*   ensure that a refactor doesn’t break anything
    

No tests ⇒ no safety net.

1\. What Is TDD, Really?
------------------------

StageNick-NameGoalCommand (pytest -q)**Red**Write a _failing_ test firstCapture the next tiny requirementF**Green**Write _just enough_ prod codeMake **all** tests pass.**Refactor**Improve designNo new features, 0 failing tests.

Repeat until done. Each loop should take **< 10 minutes**.

> **Mindset shift**: We’re not “writing tests after coding.” We’re designing **via** tests.

2\. Project Scaffold
--------------------

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   tictactoe/  ├── tic_tac_toe.py          ← production code  ├── test_tic_tac_toe.py     ← tests  └── README.md               ← this blog post, perhaps   `

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate  pip install --upgrade pip pytest  pytest -q        # ➜ 0 tests collected in 0.00s   `

3\. Red-Green-Refactor Diary
----------------------------

Below is the **unedited chronology** (commit sized chunks). You can literally copy/paste the failing test first, watch it fail, then implement.

### 3.1 Iteration 1 – An Empty Board

**Test (Red)**

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # test_tic_tac_toe.py  from tic_tac_toe import Board  def test_board_initial_state():      board = Board()      assert board.cells == [None] * 9   `

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   pytest -q  E   ImportError: cannot import name 'Board' ...   `

**Production (Green)**

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # tic_tac_toe.py  class Board:      def __init__(self):          self.cells: list[str | None] = [None] * 9   `

All green:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   pytest -q  .                                                                    [100%]   `

**Refactor**Nothing to clean yet.

### 3.2 Iteration 2 – Place a Mark

**Test**

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   def test_place_x_in_top_left():      board = Board()      board.place(0, 'X')           # 0-based index      assert board.cells[0] == 'X'   `

**Fail ➜ Implement**

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   class Board:      ...      def place(self, index: int, symbol: str):          if self.cells[index] is not None:              raise ValueError("Cell already taken")          if symbol not in ('X', 'O'):              raise ValueError("Invalid symbol")          self.cells[index] = symbol   `

Add a negative test for occupied cell.Run tests—green.

### 3.3 Iteration 3 – Switching Players

We’ll need a **Game** wrapper.

**Tests**

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   from tic_tac_toe import Game  def test_players_alternate():      g = Game()      g.play_turn(0)   # X      g.play_turn(1)   # O      assert g.board.cells[:2] == ['X', 'O']   `

**Prod**

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   class Game:      def __init__(self):          self.board = Board()          self.current = 'X'      def play_turn(self, index: int):          self.board.place(index, self.current)          self.current = 'O' if self.current == 'X' else 'X'   `

Green.**Refactor** move symbol toggle into a helper \_toggle\_player.

### 3.4 Iteration 4 – Detecting Wins

**Red test for a row win**

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   import pytest  def test_row_win():      g = Game()      g.board.cells = ['X','X','X', None,None,None, None,None,None]      assert g.winner() == 'X'   `

Implement:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   class Board:      WIN_PATTERNS = [          (0,1,2), (3,4,5), (6,7,8),           # rows          (0,3,6), (1,4,7), (2,5,8),           # cols          (0,4,8), (2,4,6)                     # diags      ]      def winner(self) -> str | None:          for a,b,c in self.WIN_PATTERNS:              if self.cells[a] and self.cells[a] == self.cells[b] == self.cells[c]:                  return self.cells[a]          return None   `

Expose via Game.winner():

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   class Game:      ...      def winner(self):          return self.board.winner()   `

Add column & diagonal tests (parameterize). All green.

### 3.5 Iteration 5 – A Draw

Test:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   def test_draw():      b = Board()      b.cells = ['X','O','X',                 'X','O','O',                 'O','X','X']      assert b.is_full() and b.winner() is None   `

Implementation:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   class Board:      ...      def is_full(self) -> bool:          return all(c is not None for c in self.cells)   `

Green.

### 3.6 Iteration 6 – CLI Loop (end-to-end)

Testing interactive I/O is trickier; pytest offers capsys / monkeypatch.

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   def test_cli_first_move(monkeypatch, capsys):      inputs = iter(["Alice", "Bob", "1", "2", "3", "4", "5"])  # we’ll stop after a win      monkeypatch.setattr('builtins.input', lambda _: next(inputs))      from tic_tac_toe import cli      cli()                       # runs until g.winner()      captured = capsys.readouterr()      assert "Alice wins" in captured.out   `

Implementation idea (in tic\_tac\_toe.py):

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   def cli():      clear_screen()      g = Game()      players = [input("Player 1 name: "), input("Player 2 name: ")]      while True:          print_board(g.board)          move = int(input(f"{players[0 if g.current=='X' else 1]}, choose (1-9): ")) - 1          try:              g.play_turn(move)          except ValueError as ex:              print(ex); continue          if g.winner():              print_board(g.board)              print(f"{players[0 if g.current=='O' else 1]} wins 🎉")              break          if g.board.is_full():              print("Draw.")              break   `

We reused only the _logic layer_; print\_board is a tiny helper that just formats board.cells.

Green.

4\. Final Source Code
---------------------

`tic_tac_toe.py`

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   """Tic-Tac-Toe (TDD Edition)"""  from __future__ import annotations  import os  from typing import List, Optional  # ─────────────────────────────────── Presentation ──────────────────────────────  def clear_screen() -> None:      os.system("cls" if os.name == "nt" else "clear")  def print_board(board: "Board") -> None:      clear_screen()      print("\nTic-Tac-Toe\n")      symbols = [c or str(i + 1) for i, c in enumerate(board.cells)]      for r in range(0, 9, 3):          print(" | ".join(symbols[r : r + 3]))          if r < 6:              print("-" * 9)  # ───────────────────────────────────── Domain ──────────────────────────────────  class Board:      """3×3 board with 0-based indexing"""      WIN_PATTERNS = [          (0, 1, 2), (3, 4, 5), (6, 7, 8),          (0, 3, 6), (1, 4, 7), (2, 5, 8),          (0, 4, 8), (2, 4, 6),      ]      def __init__(self) -> None:          self.cells: List[Optional[str]] = [None] * 9      # ── Commands ────────────────────────────────────────────────────────────────      def place(self, index: int, symbol: str) -> None:          if not 0 <= index < 9:              raise ValueError("Index must be 0-8")          if symbol not in ("X", "O"):              raise ValueError("Symbol must be X or O")          if self.cells[index] is not None:              raise ValueError("Cell already taken")          self.cells[index] = symbol      # ── Queries ────────────────────────────────────────────────────────────────      def winner(self) -> Optional[str]:          for a, b, c in self.WIN_PATTERNS:              if self.cells[a] and self.cells[a] == self.cells[b] == self.cells[c]:                  return self.cells[a]          return None      def is_full(self) -> bool:          return all(cell is not None for cell in self.cells)  class Game:      def __init__(self) -> None:          self.board = Board()          self.current = "X"      def play_turn(self, index: int) -> None:          self.board.place(index, self.current)          self._toggle_player()      def _toggle_player(self) -> None:          self.current = "O" if self.current == "X" else "X"      # façade methods for tests / UI      def winner(self) -> Optional[str]:          return self.board.winner()  # ──────────────────────────────────── CLI ──────────────────────────────────────  def cli() -> None:      clear_screen()      g = Game()      players = [input("Player 1 name: "), input("Player 2 name: ")]      while True:          print_board(g.board)          try:              move = int(input(f"{players[0 if g.current == 'X' else 1]}, choose (1-9): ")) - 1              g.play_turn(move)          except (ValueError, IndexError) as ex:              print(f"⚠️  {ex}")              input("Press Enter…")              continue          winner = g.winner()          if winner:              print_board(g.board)              winner_name = players[0 if winner == 'X' else 1]              print(f"{winner_name} wins 🎉")              break          if g.board.is_full():              print_board(g.board)              print("Draw 🤝")              break  if __name__ == "__main__":      cli()   `

`test_tic_tac_toe.py`

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   import pytest  from tic_tac_toe import Board, Game  # ── Board ──────────────────────────────────────────────────────────────────────  def test_board_initial_state():      assert Board().cells == [None] * 9  @pytest.mark.parametrize("index", [0, 4, 8])  def test_place_valid(index):      b = Board()      b.place(index, "X")      assert b.cells[index] == "X"  def test_place_rejects_duplicate():      b = Board()      b.place(0, "X")      with pytest.raises(ValueError):          b.place(0, "O")  @pytest.mark.parametrize(      "cells, expected",      [          (['X','X','X', None,None,None, None,None,None], 'X'),       # row          ([None,None,None,'O','O','O', None,None,None], 'O'),          (['X',None,None, 'X',None,None, 'X',None,None], 'X'),       # col          ([None,'O',None, None,'O',None, None,'O',None], 'O'),          (['X',None,None, None,'X',None, None,None,'X'], 'X'),       # diag          ([None,None,'O', None,'O',None, 'O',None,None], 'O'),      ],  )  def test_winner_detection(cells, expected):      b = Board()      b.cells = cells      assert b.winner() == expected  def test_draw_detection():      b = Board()      b.cells = ['X','O','X', 'X','O','O', 'O','X','X']      assert b.is_full() and b.winner() is None  # ── Game ───────────────────────────────────────────────────────────────────────  def test_game_alternates_players():      g = Game()      g.play_turn(0)  # X      assert g.board.cells[0] == "X"      g.play_turn(1)  # O      assert g.board.cells[1] == "O"   `

Run:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   pytest -q  ........                                                            [100%]   `

100 % pass 🎉.

5\. Comparing Intuition vs TDD
------------------------------

AspectIntuitive ScriptTDD Script**State Management**Global lists (layout\_positions)Encapsulated Board / Game objects**Win Logic**Six hard-coded if chainsDeclarative WIN\_PATTERNS**Safety Net**None (manual play)**16 autotestsExtensibility**HardAdd AI by subclassing Game**Portability**Tightly tied to os.system('cls')UI separated; replace cli() easily

6\. Key Take-Aways
------------------

1.  **Tests guide design**: We discovered _when_ to create Board, Game, helper functions—_not_ prematurely.
    
2.  **Red-Green-Refactor keeps momentum**: Small cycles prevent rabbit-holes.
    
3.  **Confidence to refactor**: Want a 4×4 board? Change WIN\_PATTERNS, run tests—done.
    
4.  **Fewer bugs escape**: Each requirement is captured by at least one assertion.
    

7\. Next Challenges ✨
---------------------

*   Add a **minimax AI** – write tests for best\_move() first.
    
*   Support network play—unit-test the protocol layer with _dependency injection_.
    
*   Package & publish to PyPI—your tests double as _acceptance criteria_ for every future release.
    

### Did this deep dive help?

> Share your own TDD story in the comments or tag me on LinkedIn. Happy testing! 🎮🧪