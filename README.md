# Tetris-Javafx
CSC207 Assignment 2: Tetris
Due: November 18, 8 p.m.
1 Introduction
This assignment is inspired by one originally designed by Nick Parlante. Thank you to him for sharing.
This assignment’s major learning goal is to master the basics of a model-viewcontroller system, and to create a fun game with a graphical user interface
(GUI) in the process. More specifically, you will build up a set of classes for the
game of Tetris. The assignment will emphasize using encapsulation to divide a
big problem into many independently testable problems. The first part of the
assignment sets up Pieces and a Board for the game of Tetris. The second part
ties the model to the front end (i.e. to the View). For reasons that will become
clear later, there is a theme of efficiency in this design. We are not just writing
classes that implement Tetris. We are writing classes that implement Tetris
quickly.
If you don’t know what Tetris is, don’t worry ... you can play the game online at this URL: https://www.freetetris.org/game.php. If you play long
enough, you may discover the Tetris effect (see https://en.wikipedia.org/
wiki/Tetris_effect). So you might not want to overdo it :).
In the first part of this specification, we provide an overview of the various
components of the software you will be extending. The starter code contains
some additional information about the assignment, and the relevant classes,
as well. The major classes that you will find in the starter code include a
TetrisPiece and TetrisPoint class, a TetrisBoard and TetrisModel class,
as well as three kinds of View: a TetrisView, a LoadView and a SaveView.
You will also find a class for an AutoPilot, which will be used to put the game
in autopilot mode. This implements the TetrisPilot interface.
In the second part of this specification, we’ll summarize the specific methods
you are expected to implement.
1
Figure 1: The Seven Tetris pieces and their rotations. From https://tetris.
wiki/
2 The Major Tetris Classes
2.1 TetrisPiece
There are seven pieces in standard Tetris, and these are encoded in the TetrisPiece
class. A standard piece is composed of four blocks (defined as TetrisPoints in
the starter code). The two “L” and “S” pieces are mirror images of each other,
but we’ll be thinking of them as similar but distinct pieces. A picture of all of
these pieces, and their various rotated versions, can be found in Figure 1.
A piece can be rotated 90 degrees counter-clockwise to yield another piece.
Enough rotations get you back to the original piece. For example, rotating an
“S” shaped piece twice brings you back to the original state. Rotating an “L”
shaped piece four times brings you back to the original state.
In our abstraction, a single TetrisPiece represents a piece in a single rotation.
This means there will be a total of four TetrisPieces required to represent the
possible orientations of the “L” shaped piece, and 2 to represent the orientations
2
of an “S”. Some of a TetrisPiece’s attributes and methods are described below.
2.1.1 The Body
A TetrisPiece consists of TetrisPoints which make up the “body” of the piece.
Each TetrisPiece has its own coordinate system with the (0,0) origin located in
the lower left hand corner of the rectangle that encloses the body. The coordinates of TetrisPoints in the body will all be relative to the origin of the piece.
So, the four TetrisPoints that make up the square piece will have coordinates:
[(0, 0),(0, 1),(1, 0),(1, 1)].
Notice that not all pieces actually have a TetrisPoint at (0,0). For example, the
body of the right ‘dog’ (in the last column of the 4th row in Figure 1) has the
body: [(0, 1),(0, 2),(1, 0),(1, 1)].
A piece is completely defined by its body. All its other qualities, such as its
height and width, can be computed from the body. The right dog described
above a width of 2 and height of 3.
2.1.2 The “Lowest Y Values”
It is useful to store some data about each TetrisPiece to enable quick play. More
specifically, we will define the “lowestYVals” of a given TetrisPiece as an int[]
array that is as long as the piece is wide. In this array we will store the lowest
y value for each x value in the TetrisPiece’s coordinate system. We will use this
array to help us determine where a piece will land once it has been placed. The
“lowestYVal” of the piece with coordinates [(0, 1),(0, 2),(1, 0),(1, 1)] is (1, 0).
We will assume that pieces do not have holes in them, meaning that for every
x in the piece coordinate system there will be at least one TetrisPoint in the
piece for that x.
2.1.3 Rotations
To make rotations of pieces fast, we will pre-compute all rotations for each
piece at the start of the game. Each TetrisPiece will then include a reference
to another TetrisPiece that represents the same TetrisPiece after a clockwise
rotation. This will mean that when given a TetrisPiece, it will then be easy to
locate the TetrisPiece after a rotation is applied. This should allows the client
to obtain rotations of each Piece in constant time, effectively.
To implement the rotation strategy we will first want to make a single TetrisPiece
to represent each of the 7 pieces before rotations are applied. Each of these pieces
will then be joined to other pieces in a little circular linked list. The client will
use the computeNextRotation() method to iterate through all the rotations of
a single TetrisPiece. The arrays for each piece will allocated the first time the
client calls getP ieces(). Initializing this data structure the first time it is used
3
is a trick is called ”lazy evaluation”; we build the pieces only at the time they
are actually needed.
You will likely want private methods behind the scenes to compute the rotations of each piece and to assemble all the rotations into a linked list. Notice
that the width, height and lowestYVals attributes will be affected when making
new pieces and when computation rotations.
2.2 The Board
The TetrisBoard class:
1. Stores the current state of a Tetris board;
2. Supports the placement of pieces and checks for errors in placement.
3. Facilitates “undo” operations, so that the board can be restored to a valid
state should a move requested be invalid or impossible. Some of the code
for to support this functionality has been provided for you.
4. Does all of the above quickly.
The board’s most obvious attribute is the tetrisGrid, which is a 2-d array
of booleans that stores which spots are filled. The lower left corner is at coordinate (0,0). The X value increases to the right and Y increases upwards.
The placeP iece() operation supports adding a piece into the grid, and the
clearRows() operation will clear filled rows in the grid and shift all the other
Tetris blocks down.
2.2.1 Constructor
The constructor initializes a new empty board. The board may be any size, but
a standard Tetris board is 10 wide and 20 high. The TetrisModel additionally
allocates a few (i.e. 4) extra rows at the top of the grid to allow extra space
for new pieces to fall into play. The constructor also allocates space for a
backupGrid that is used to facilitate “undo” operations.
2.2.2 rowCounts and colCounts
The TetrisBoard’s rowCounts attribute stores how many filled spots there are
in each row. This allows the placeP iece() operation to detect efficiently if the
placement has caused a row to become filled (and if clearRows() must be called).
The colCounts attribute stores the number of grid cells filled in a given column
(i.e. the height of the filled cells in a column). This height will tell you the index
of the open spot just above the last filled spot in the column. The colCounts
array allows the placementHeight() operation to determine where a piece will
come to rest if it is dropped or lands in a particular column.
4
2.2.3 int placePiece(piece, x, y)
This method takes a piece and an (x,y) coordinate and attempts to place the origin (i.e. the lower-left corner) of the piece at that location on the board. There
are lots of ways this can go wrong. It’s possible that the new placement falls
outside of the board or that overlaps spots that are already filled. If part of the
piece falls out of bounds when placed, we will return ADD OUT BOUNDS. If
the piece overlaps already filled spots, return ADD BAD. Note that an invalid
placement may put the board in an invalid state. The client should be able to
return the board to a valid, pre-placement state with a call to ”undo()”. If the
placement is good, the method will return ADD OK. And if the placement
is both good and it caused at least one row to become filled, the method will
return ADD ROW F ILLED.
2.2.4 clearRows()
This method should delete each row that is filled all the way across, causing all
the blocks in the grid that are above to to shift down. There may be multiple
filled rows and they may not be adjacent.
A fast solution will do the shift in one pass, but it’s ok if your code needs
to make multiple passes. Remember that you have the colCounts array and
so you know the maximum filled height of all the columns. This should help
you avoid copying of empty space at the top of the board. Also, note that the
colCounts array will need to be recomputed after row clearing. The new value
for each column will be lower than the old value after clearing (by at least one
and possibly more).
2.2.5 int placementHeight(piece, x)
This method computes the y value where the origin (0,0) of a piece will come to
rest if it is dropped or placed in the given column. The method should use the
colCounts array and the lowestY V als to compute the resting y value quickly.
The method assumes the piece falls straight down – there will be no moving of
the the piece around things during a drop.
2.2.6 undo()
Some operations will ask you to experiment with adding different pieces. To
support this, the board implements undo() and some code for this has been
supplied. Note that he board has a “committed” state which is either true or
false. This will help us keep track as to whether a move can be undone or not.
More specifically, the way undo will operate is as follows:
1. The board will start in a “committed” state, meaning that the committed
attribute will be true.
5
2. The client may do a single placeP iece() operation at this point; this should
set committed to false. The board must be in the committed state before
placeP iece() is called, so it is not possible to call placeP iece() twice in a
row.
3. The client may then (optionally) do a single clearRows() operation, which
will maintain committed as ’false’.
4. At this point, the client may do an undo() operation which will return
the board to its original “committed” state. It will set the committed
attribute to true when this is done.
5. Alternately, the client may do a commit() operation which will keep the
board in its altered state and set the committed attribute back to true.
This is going forwards.
6. The client must either undo() or commit() before doing another placeP iece().
2.3 The TetrisView
A TetrisView provides a graphical way for you to see the TetrisModel in action, and for you to change the various parameters of the game. You won’t be
able to see it load until you complete some of the model methods tho, like the
TetrisPiece constructor.
The TetrisView contains a variety of components, which you will have to link
to the model. These components include:
1. The newButton, which a user should be able to use to start a new Tetris
game. When this is clicked, the various attributes related to a given game
(e.g. the TetrisBoard) will have to be wiped clean.
2. The startButton and stopButton, which a user should be able to use to
pause and play a given Tetris game. When the stop button is pressed, all
the action should stop! When the startButton is pressed, the action can
resume.
3. The saveButton. This should be configured such that a SaveView pops
up when pressed. The SaveView contains a textbox that users can use
to input the name of the file to be saved. A saved file corresponds to a
serialized TetrisModel.
4. The loadButton. This should be configured such that a LoadView pops
up when pressed. The LoadView contains a listView that users can use to
select the name of the file to be load into play. A loaded file corresponds
to a deserialized TetrisModel.
5. The slider. This should be configured such that is alters the pace of the
game. The more the slider is moved to the right, the faster the game! The
more it is moved to the left, the slower the game.
6
6. The borderPane. This is where the TetrisBoard and TetrisPieces will be
displayed. You will want to configure the pane such that it responds to
key presses from the user. The user should be able to Rotate a given
piece, move it Right or Left, or Drop it. Use whatever keys you feel most
comfortable with to make these actions take place.
2.4 The AutoPilot (an optional task!)
The last thing you can work on, if your time allows, is an autopilot for Tetris!
The autopilot will allow the computer to auto-play the pieces as they fall. We
have provided a random autopilot that selects moves at random for you, but
you are more than welcome, and in fact encouraged, to change this.
The AutoPilot interface defines the bestMove() method; this will return what
it thinks is the best available move for a given piece and board. The random
strategy, although it works, is obviously not the best! You might instead want
to try looking ahead at the moves that are possible from a given position, and
then selecting the move that looks the most promising.
What makes a promising looking board? That’s for you to decide. If you
can translate your understanding of a good looking board into a number, you
can encode this using the evaluateBoard method in the AutoPilot class. Try
generating several boards that might lie ahead, given a range of possible moves.
Evaluate each and every one of these possible boards using evaluateBoard and
pick the move that leads you to the board that looks ”best”.
There are of course mountains of strategies you might use to look ahead! For
now, you don’t need to get too complicated. But if you’re interested, you might
look up some strategies to search through all of the possible boards that lie
ahead, like depth first search or breadth first search. Perhaps you can find an
even better strategy, too!
3 To Dos
Download the assignment files from Quercus. Modify TetrisPiece.java, Board.java,
LoadView.java, SaveView.java, TetrisView.java and AutoPilot.java appropriately so that they complete the tasks outlined in this document on the
next page. As you code you:
1. may add more private methods (do not remove any of the provided methods though!)
2. may add additional classes
3. use static and instance variables/methods as appropriate, with appropriate levels of protection
7
4. must appropriately document your code
5. must appropriately use inheritance, composition, and polymorphism
You will be completing the following methods in the TetrisPiece class:
1. The TetrisPiece constructor, which take in an array of TetrisPoints and
store a copy of these points. It will also define the piece’s “lowestYVals”.
For each x value in the body of a piece, the lowestYVals will contain the
lowest y value in the body for that x.
2. makeFastRotations, which will be given a piece (the “root” rotation), and
will compute all the other rotations of that piece. It will then links all
the rotations of a piece together in a circular list so they can be easily
accessed via each piece’s ”next” attribute.
3. equals, which will returns true if two pieces have bodies that contain the
same TetrisPoints.
In the TetrisBoard class you will complete:
1. placementHeight, which will take in an piece and an x coordinate and
returns the y value where the piece would come to rest if it were dropped
straight down at that x.
2. placePiece, which will take in an piece and an x and y coordinate. See the
comments in the starter code for more information.
3. clearRows, which will deletes rows that are filled all the way across and
move all the blocks above the cleared row down.
In the LoadView class you will complete:
1. The getFiles method, which will populate a listView of files to load with
all the ”.ser” files that exist in the ”boards” directory.
2. selectBoard, which will loads the board file selected in the listView and
update the GUI to display the name of the board file that has been loaded.
In the SaveView class you will complete:
1. The saveBoard method, which will save a .ser files with the current game
state to the boards directory so that it can be loaded later.
In the TetrisView class you will complete:
1. The initUI method. Most of this has been written for you, but you are
asked to link the various GUI components to the actions that they should
be listening for. See the code for details here.
In the AutoPilot class you may optionally complete:
8
1. The bestMove method. A default implementation exists here, which is
really bad! You can try to improve this with whatever time allows. If
you’re especially eager, you might try looking at board configurations
that lie a few moves ahead when you’re making your move selection. You
might be able to reach some good looking board configurations, and some
that are not so good looking! Can you locate a move that is associated
with a ”good” looking board?
4 Deliverables
Download the assignment files from Quercus. Modify TetrisPiece.java, TetrisBoard.java,
LoadView.java, SaveView.java, TetrisView.java and AutoPilot.java appropriately so that they complete the tasks outlined above.
Submit your modified TetrisPiece.java, TetrisBoard.java, LoadView.java,
SaveView.java, TetrisView.java and AutoPilot.java. In addition, submit
a file called acknowledgment.txt in which you write the following words:
The code I am submitting is my own, and has been written by no one other
than me.
How to submit: If you submit before you have used all of your grace days, you
will submit your assignment using MarkUs. It is your responsibility to include
all necessary files in your submission. You can submit a new version of any file
at any time, though the lateness penalty applies if you submit after the deadline. For the purposes of determining the lateness penalty, the submission time
is considered to be the time of your latest submission.
Extra Information
Clarification Page: Important corrections (hopefully few or none) and clarifications to the assignment will be posted on the Assignment 2 Clarification page
on Quercus. You are responsible for monitoring the Assignment 2 Clarification
page.
Questions: Questions about the assignment should be asked on Piazza.1
If you
have a question of a personal nature, please email the course email, placing [A2]
in the subject header.
Marking Criteria
We will test your code electronically. If your code fails all of the tests, you will
receive zero marks. It’s up to you to create test cases to test your code
1https://piazza.com/class/l6uzmwn9y342ii.
9
- that’s part of the assignment!
Your code will not be evaluated for partial correctness; it either works or it
doesn’t. It is your responsibility to hand in something that passes at least some
of the tests that were given to you with the starter code.
The marking criteria for this portion of your work will be as follows:
Passing Tests released with the assignment will be worth 10%
TetrisPiece tests will be worth 25%
TetrisBoard tests will be worth 30%
LoadView tests will be worth 10%
SaveView tests will be worth 10%
TetrisView tests will be worth 15%
Implementing the AutoPilot is optional and will not count toward your mark.
Have fun and GOOD LUCK!!
10
