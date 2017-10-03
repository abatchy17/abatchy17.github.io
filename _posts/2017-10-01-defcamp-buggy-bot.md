---
layout: post
title: "[DefCamp CTF Qualification 2017] Buggy Bot (Misc 400)"
date: 2017-10-01 11:00:00
share: true
comments: true
description: Writeup for Buggy Bot!
tags: [DefCamp CTF Qualifications 2017]
---

## Buggy Bot

The python [code](https://dctf.def.camp/quals-2017-kalskflsafkl/chess.py) provided allows you to make a single move then it makes some predefined moves. The goal was a bit confusing to me at first as I wasn't sure if they wanted the position of the king after the first move only (assuming it survided) or its final position regardless. It was the latter.

I tweaked the code a bit so that it tries all the possible (wrong) moves that would matter, which are the king's only, moving other pieces is irrelevant. So basically:

1. Make king move from e1 to X ([a-h][1-8]).
2. Let bot make moves.
3. If king survived after those moves, add its position to a set list.

This sloppy code does it:

```python
#/usr/bin/python2.7
column_reference = "a b c d e f g h".split(" ")
EMPTY_SQUARE = " "
row_letters = "abcdefgh"

king_moves = []
position = ''
dest = ''
solution = set([])

class Model(object):
    def __init__(self):
        self.board = []
        pawn_base = "P "*8
        white_pieces =  "R N B Q K B N R"
        white_pawns = pawn_base.strip() 
        black_pieces = white_pieces.lower()
        black_pawns = white_pawns.lower()
        self.board.append(black_pieces.split(" "))
        self.board.append(black_pawns.split(" "))
        for i in range(4):
            self.board.append([EMPTY_SQUARE]*8)
        self.board.append(white_pawns.split(" "))
        self.board.append(white_pieces.split(" "))
 
    def move(self, start,  destination):
        for c in [start, destination]:
            if c.i > 7 or c.j > 7 or c.i <0 or c.j <0:
                return
        if start.i == destination.i and start.j == destination.j:
            return
 
        if self.board[start.i][start.j] == EMPTY_SQUARE:
            return
             
        f = self.board[start.i][start.j]
        self.board[destination.i][destination.j] = f
        self.board[start.i][start.j] = EMPTY_SQUARE
 
 
class BoardLocation(object):
    def __init__(self, i, j):
        self.i = i
        self.j = j
         
 
class View(object):
    def __init__(self):
        pass
    def display(self,  board):
        print("%s: %s"%(" ", column_reference))
        print("-"*50)
        for i, row in enumerate(board):
            row_marker = 8-i
            print("%s: %s"%(row_marker,  row))
         
 
class Controller(object):
    def __init__(self):
        self.model = Model()
        self.view = View()
     
    def run(self):
        global solution
        move = position
        start,  destination = self.parse_move(move)
        self.model.move(start, destination)
        start,  destination = self.parse_move("a2-b2")
        self.model.move(start, destination)
        start,  destination = self.parse_move("b2-c2")
        self.model.move(start, destination)
        start,  destination = self.parse_move("c2-d2")
        self.model.move(start, destination)
        start,  destination = self.parse_move("e2-f2")
        self.model.move(start, destination)
        start,  destination = self.parse_move("f2-g2")
        self.model.move(start, destination)
        start,  destination = self.parse_move("g2-h2")
        self.model.move(start, destination)
        start,  destination = self.parse_move("h2-a1")
        self.model.move(start, destination)
        start,  destination = self.parse_move("a1-b1")
        self.model.move(start, destination)
        start,  destination = self.parse_move("b1-c1")
        self.model.move(start, destination)
        start,  destination = self.parse_move("c1-d1")
        self.model.move(start, destination)
        start,  destination = self.parse_move("e1-f1")
        self.model.move(start, destination)
        start,  destination = self.parse_move("f1-g1")
        self.model.move(start, destination)
        start,  destination = self.parse_move("g1-h1")
        self.model.move(start, destination)
        start,  destination = self.parse_move("h1-a3")
        self.model.move(start, destination)
        start,  destination = self.parse_move("a3-b3")
        self.model.move(start, destination)
        start,  destination = self.parse_move("b3-c3")
        self.model.move(start, destination)
        start,  destination = self.parse_move("c3-d3")
        self.model.move(start, destination)
        start,  destination = self.parse_move("e3-f3")
        self.model.move(start, destination)
        start,  destination = self.parse_move("f3-g3")
        self.model.move(start, destination)
        start,  destination = self.parse_move("g3-h3")
        self.model.move(start, destination)
        start,  destination = self.parse_move("h3-a4")
        self.model.move(start, destination)
        start,  destination = self.parse_move("a4-b4")
        self.model.move(start, destination)
        start,  destination = self.parse_move("b4-c4")
        self.model.move(start, destination)
        start,  destination = self.parse_move("c4-d4")
        self.model.move(start, destination)
        start,  destination = self.parse_move("e4-f4")
        self.model.move(start, destination)
        start,  destination = self.parse_move("f4-g4")
        self.model.move(start, destination)
        start,  destination = self.parse_move("g4-h4")
        self.model.move(start, destination)
        start,  destination = self.parse_move("h4-a5")
        self.model.move(start, destination)
        start,  destination = self.parse_move("a5-b5")
        self.model.move(start, destination)
        start,  destination = self.parse_move("b5-c5")
        self.model.move(start, destination)
        start,  destination = self.parse_move("c5-d5")
        self.model.move(start, destination)
        start,  destination = self.parse_move("e5-f5")
        self.model.move(start, destination)
        start,  destination = self.parse_move("f5-g5")
        self.model.move(start, destination)
        start,  destination = self.parse_move("g5-h5")
        self.model.move(start, destination)
        start,  destination = self.parse_move("h5-a6")
        self.model.move(start, destination)
        start,  destination = self.parse_move("a6-b6")
        self.model.move(start, destination)
        start,  destination = self.parse_move("b6-c6")
        self.model.move(start, destination)
        start,  destination = self.parse_move("c6-d6")
        self.model.move(start, destination)
        start,  destination = self.parse_move("e6-f6")
        self.model.move(start, destination)
        start,  destination = self.parse_move("f6-g6")
        self.model.move(start, destination)
        start,  destination = self.parse_move("g6-h6")
        self.model.move(start, destination)
        start,  destination = self.parse_move("h6-a7")
        self.model.move(start, destination)
        start,  destination = self.parse_move("a7-b7")
        self.model.move(start, destination)
        start,  destination = self.parse_move("b7-c7")
        self.model.move(start, destination)
        start,  destination = self.parse_move("c7-d7")
        self.model.move(start, destination)
        start,  destination = self.parse_move("e7-f7")
        self.model.move(start, destination)
        start,  destination = self.parse_move("f7-g7")
        self.model.move(start, destination)
        start,  destination = self.parse_move("g7-h7")
        self.model.move(start, destination)
        
        for i, row in enumerate(self.model.board):
            row_marker = 8-i
            if 'K' in row:
                print "Move: " + position + "({1}{0})".format(row_marker, row_letters[row.index('K')])
                solution.add("{1}{0}".format(row_letters[row.index('K')],row_marker ))

        
        # Print board if needed
        # self.view.display(self.model.board)
            
    def parse_move(self, move):
         
        s, d = move.split("-")
        i = 8- int(s[-1])
        j = column_reference.index(s[0])
        start = BoardLocation(i, j)
         
        i =  8- int(d[-1])
        j= column_reference.index(d[0])
        destination = BoardLocation(i, j)
 
        return start,  destination
         
 
if __name__=="__main__":
    for j in '12345678':
        for i in 'abcdefgh':
            king_moves.append('e1-' + i + j)
            dest = i+j
            position = 'e1-' + i + j
            C = Controller()
            C.run()
    solution  = sorted(solution)
    out = ''
    for position in solution:
        out = out + position[::-1] + ";"
    print out
```

```console
abatchy@ubuntu:~/Desktop/tmp$ python a.py
Move: e1-e1(d3)
Move: e1-f1(d3)
Move: e1-a2(d2)
Move: e1-e2(d1)
Move: e1-e3(d4)
Move: e1-f3(d4)
Move: e1-g3(d4)
Move: e1-h3(d4)
Move: e1-a4(d4)
Move: e1-b4(d4)
Move: e1-c4(d4)
Move: e1-d4(d4)
Move: e1-e4(d5)
Move: e1-f4(d5)
Move: e1-g4(d5)
Move: e1-h4(d5)
Move: e1-a5(d5)
Move: e1-b5(d5)
Move: e1-c5(d5)
Move: e1-d5(d5)
Move: e1-e5(d6)
Move: e1-f5(d6)
Move: e1-g5(d6)
Move: e1-h5(d6)
Move: e1-a6(d6)
Move: e1-b6(d6)
Move: e1-c6(d6)
Move: e1-d6(d6)
Move: e1-e6(d7)
Move: e1-f6(d7)
Move: e1-g6(d7)
Move: e1-h6(d7)
Move: e1-a7(d7)
Move: e1-e7(h7)
Move: e1-a8(a8)
Move: e1-b8(b8)
Move: e1-c8(c8)
Move: e1-d8(d8)
Move: e1-e8(e8)
Move: e1-f8(f8)
Move: e1-g8(g8)
Move: e1-h8(h8)
d1;d2;d3;d4;d5;d6;d7;h7;a8;b8;c8;d8;e8;f8;g8;h8
```

The flag is `DCTF{sha256(positions)}` = `DCTF{1bdd0a4382410d33cd0a0bf0e8193345babc608ea0ddd83dccbcb4763d67c67b}`.

\- Abatchy