---
date: 2019-05-25
title: Labo IDP - Galaxies
author: Dries Kennes
font: DejaVu Sans
monofont: DejaVu Sans Mono
---

# IDPGalaxies

**IDP solver for the Galaxies logic puzzle.**

Copyright © 2019 Dries Kennes <http://dries007.net> & Laurens VDW \
All rights reserved.

The rules of the puzzle:

1) Every region should have two-way rotational symmetry,
2) should contain exactly one dot which is in its centre, and
3) should contain no lines separating two of its own squares from each other.
4) No black holes.

## How the Python wrapper works

_Magic of the darkest kind,_ otherwise known as hardcoding. Big thanks to Pycharm and it's column selection mode for making this much less work. 

The Python "wrapper" script creates the IDP file and runs it. The output of IDP is then parsed and virified. The output is appended to the IDP file as a comment, along with the parsed model grids. **Warning: It overwrites the galaxies.idp file every run!**

This wrapper was created because looking at a set of coordinates to verify the puzzle is not fun. Spending 4 hours creating a script to draw the right Unicode box drawing characters on the other hand, that's my idea of a great saturday night.

When it's run an interactive prompt is shown. This prompt can be used to input model output from the IDP-Web IDE thing. The script will parse it and print the output. This was used to quickly test small changes in the IDP by opening the output file in the Web-IDE and re-running it. **Put an empty line in to print the current state (grid).** The input (centers and grid size) _cannot_ be modified via the interpreter. Change the script to edit those variables. Stop it with a control-C.

The parsing is just dumb regex, but it does the job. Every line is parsed as 1 unit, so you cannot spit (wrap) lines. Unknown lines are ignored, this makes it easy to copy-past multiple models in one go.

The main function is at the bottom of the file, select/set the input there. The script comes with 4 build in example inputs, all taken from [Simon Tatham's Portable Puzzle Collection](https://www.chiark.greenend.org.uk/~sgtatham/puzzles/js/galaxies.html).

## How the IDP part works

There are comments in the template/output file, they contain info on what every IDP line does. But here is some of the key insights:

1) The grid coordinates are doubled, so everything is on the same coordinate system.
    + Walls can only appear on even coordinates.
    + Regions only have odd coordinates.
    + Centers can be on all positions.
2) Some helper relations where used:
    + `Center`: Puzzle input. Set of center coordinates. 
    + `Square`: Set of all even coordinates (regions)
    + `Touching`: Maps what squares directly touch which centers.
    + `BelongsTo`: Maps what region belongs to which center.  
    + `Reachable`: Maps what region is reachable from a square.
    + `Walls`: Puzzle output. Set of wall coordinates.
3) Dead-end walls are not allowed, this means that every wall must have 2 or more neighbours.
4) Walls must surround different regions on all sides.
5) Every `Touching` is also a `BelongsTo`.
7) `BelongsTo` is used as the "unknown" glue that ties the relations together.
8) `Walls` block `Reachable`
9) `Reachable` is defined recursively and seeded with `Touching`. 
10) Symmetry is easier than it seems, 2*center-coordinate = coordinate of mirror.

\newpage
## Examples

Every character in this grid is a center. It uses A-Z first, but will go on to 0-9 and symbols if it runs out. Using A-Z is nice because lowercase is used to indicate the region association if that option is enabled.

Note: The actual input is a coordinate list, this is just visuals.

### 5x5

    Input               Output
                1                   1
      01234567890         01234567890
     0┏━┯━┯━┯━┯━┓        0┏━┯━┯━┯━┳━┓
     1┃ ┆ A ┆ ┆ ┃        1┃a┆aAa┆a┃c┃
     2┠┄┼┄┼┄┼┄┼┄┨        2┣━┿━┿━╈━╃┄┨
     3┃ ┆B┆ ┆ ┆ ┃        3┃b┆B┆b┃c┆c┃
     4┠┄┼┄┼┄┼┄C┄┨        4┣━┿━┿━╉┄C┄┨
     5┃ ┆D┆ ┆ ┆ ┃        5┃d┆D┆d┃c┆c┃
     6┠┄┼┄┼┄┼┄┼┄┨        6┣━╈━┿━╉┄╆━┫
     7┃ ┆ ┆ ┆ ┆ ┃        7┃e┃f┆f┃c┃g┃
     8┠E┼┄┼F┼┄┼G┨        8┠E╊━╅F╄━╉G┨
     9┃ ┆H┆ ┆ ┆ ┃        9┃e┃H┃f┆f┃g┃
    10┗━┷━┷━┷━┷━┛       10┗━┻━┻━┷━┻━┛

### 7x7

    Input                   Output
                11111                   11111
      012345678901234         012345678901234
     0┏━┯━┯━┯━┯━┯━┯━┓        0┏━┯━┯━┳━┯━┳━┯━┓
     1┃ ┆ ┆ ┆ A ┆ ┆ ┃        1┃b┆b┆b┃aAa┃c┆c┃
     2┠┄┼B┼┄┼┄┼┄┼┄C┄┨        2┠┄┼B┼┄╊━┿━╉┄C┄┨
     3┃ ┆ ┆ ┆ ┆ ┆ ┆ ┃        3┃b┆b┆b┃d┆d┃c┆c┃
     4┠┄┼┄┼┄┼┄┼D┼┄┼┄┨        4┣━┿━╈━╇━╅D╄━╈━┫
     5┃ E ┆ ┆ ┆ ┆ ┆ ┃        5┃eEe┃g┆g┃d┆d┃h┃
     6┠┄┼┄┼┄┼┄┼┄┼┄┼┄┨        6┣━┿━╉┄┼┄╄━┿━╉┄┨
     7┃ ┆ ┆ ┆ ┆ ┆ ┆ ┃        7┃f┆f┃g┆g┆g┆g┃h┃
     8┠┄F┄┼┄┼┄┼┄┼┄┼┄┨        8┠┄F┄╂┄┼┄╆━┿━╉┄┨
     9┃ ┆ ┆ G ┆ ┆ ┆H┃        9┃f┆f┃gGg┃i┆i┃H┃
    10┠┄┼┄┼┄┼┄┼┄I┄┼┄┨       10┣━┿━╃┄┼┄╂┄I┄╂┄┨
    11┃ ┆ ┆ ┆ ┆ ┆ ┆ ┃       11┃g┆g┆g┆g┃i┆i┃h┃
    12┠┄┼┄┼┄┼┄┼┄┼┄┼┄┨       12┣━┿━╅┄┼┄╊━┿━╉┄┨
    13┃ J ┆ ┆ ┆ K ┆ ┃       13┃jJj┃g┆g┃kKk┃h┃
    14┗━┷━┷━┷━┷━┷━┷━┛       14┗━┷━┻━┷━┻━┷━┻━┛

\newpage
### 10x10

This is with region associations and inner walls not printed, it's too busy in such a large grid.
    
    Input                       Output
                11111111112                 11111111112
      012345678901234567890       012345678901234567890
     0┏━┯━┯━┯━┯━┯━┯━┯━┯━┯━┓      0┏━┯━┳━┯━┯━┳━┯━┯━┳━┳━┓
     1┃ A    B     C      ┃      1┃ A ┃  B  ┃  C  ┃ ┃ ┃
     2┠ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┼D┨      2┣━┿━╇━┿━┿━╋━╈━┿━╉ ╂D┨
     3┃    E              ┃      3┃    E    ┃ ┃   ┃ ┃ ┃
     4┠ ┼ ┼ ┼ ┼ ┼F┼ G ┼ ┼ ┨      4┣━┿━╈━┿━╈━╉F╂ G ╂ ╊━┫
     5┃     H          I  ┃      5┃   ┃ H ┃ ┃ ┃   ┃I┃ ┃
     6┠ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┨      6┣━╅ ╄━┿━╉ ╄━╇━┿━╉ ╂ ┨
     7┃J                  ┃      7┃J┃     ┃       ┃ ┃ ┃
     8┠ ┼ ┼K┼ ┼ ┼ L ┼ ┼ ┼M┨      8┣━╉ ┼K┼ ╂ ┼ L ┼ ╂ ╂M┨
     9┃                   ┃      9┃ ┃     ┃       ┃ ┃ ┃
    10┠N┼ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┨     10┠N╊━╈━╅ ╄━╈━╈━╅ ╊━╉ ┨
    11┃  O         P   Q  ┃     11┃ ┃O┃ ┃   ┃ ┃P┃ ┃Q┃ ┃
    12┠ ┼ ┼R┼ ┼ ┼ ┼ ┼ ┼ ┼ ┨     12┣━╇━╉R╊━╈━╃ ╊━╇━╋━╇━┫
    13┃      S      T     ┃     13┃   ┃ ┃S┃   ┃ T ┃   ┃
    14┠ U ┼ ┼ ┼ V ┼ ┼ ┼ ┼ ┨     14┠ U ╊━╇━╉ V ╊━┿━╉ ┼ ┨
    15┃     W             ┃     15┃   ┃ W ┃   ┃   ┃   ┃
    16┠ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┼ X ┨     16┣━┿━╋━┿━╉ ╆━╃ ┼ ╂ X ┨
    17┃            Y      ┃     17┃   ┃   ┃ ┃  Y  ┃   ┃
    18┠ Z ┼ ┼0┼ ┼ ┼ ┼ ┼ ┼ ┨     18┠ Z ╊━╅0╄━╉ ┼ ╆━╉ ┼ ┨
    19┃    1         2    ┃     19┃   ┃1┃   ┃   ┃2┃   ┃
    20┗━┷━┷━┷━┷━┷━┷━┷━┷━┷━┛     20┗━┷━┻━┻━┷━┻━┷━┻━┻━┷━┛

Runtime:  8 sec

\newpage
### 15x15

This is with region associations and inner walls not printed, it's too busy in such a large grid.
    
    Input                               Output
                111111111122222222223               111111111122222222223
      0123456789012345678901234567890     0123456789012345678901234567890
     0┏━┯━┯━┯━┯━┯━┯━┯━┯━┯━┯━┯━┯━┯━┯━┓    0┏━┳━┳━┯━┳━┳━┳━┯━┳━┯━┯━┯━┳━┯━┯━┓
     1┃ ┆A┆ ┆ ┆B┆ ┆ ┆ ┆ ┆ C ┆ ┆ ┆ ┆ ┃    1┃ ┃A┃   ┃B┃ ┃   ┃   C   ┃     ┃
     2┠┄┼┄┼┄D┄┼┄┼E┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┨    2┠ ╄━╉ D ╊━╉E╂ ┼ ╊━┿━╈━┿━╃ ┼ ┼ ┨
     3┃ ┆ ┆ ┆ ┆ ┆ ┆ F ┆ G ┆ ┆ ┆H┆ ┆ ┃    3┃   ┃   ┃ ┃ ┃ F ┃ G ┃    H    ┃
     4┠┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┨    4┠ ┼ ╄━┿━╃ ╄━╉ ┼ ╊━┿━╉ ┼ ┼ ╆━┿━┫
     5┃ ┆ ┆ I ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┃    5┃     I     ┃   ┃   ┃     ┃   ┃
     6┠┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┨    6┣━╅ ╆━╈━╅ ┼ ╊━╈━╃ ┼ ╄━┿━┿━╃ ┼ ┨
     7┃ ┆ ┆J┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┃    7┃ ┃ ┃J┃ ┃   ┃ ┃               ┃
     8┠┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄K┄┼┄┼┄┼┄┨    8┠ ╊━╇━╉ ╊━╅ ╂ ╂ ┼ ┼ ┼ K ┼ ┼ ┼ ┨
     9┃L┆ ┆ ┆ ┆ ┆ ┆M┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┃    9┃L┃   ┃ ┃ ┃ ┃M┃               ┃
    10┠┄┼┄N┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┨   10┠ ╂ N ╂ ╂ ╊━╉ ╂ ┼ ╆━┿━╈━╅ ┼ ╆━┫
    11┃ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆O┆ ┆ ┆ ┃   11┃ ┃   ┃ ┃ ┃ ┃ ┃   ┃   ┃O┃   ┃ ┃
    12┠┄┼┄┼┄┼P┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼Q┨   12┣━╋━┿━╉P╂ ╂ ╄━╋━╈━╃ ┼ ╄━╇━┿━╉Q┨
    13┃R┆ ┆ ┆ ┆ ┆ ┆ ┆S┆ ┆ ┆ ┆ ┆ ┆ ┆ ┃   13┃R┃   ┃ ┃ ┃   ┃S┃           ┃ ┃
    14┠┄┼┄┼┄┼┄┼T┼┄U┄┼┄┼┄┼┄┼V┼┄┼┄┼┄┼┄┨   14┣━╃ ┼ ╂ ╂T╂ U ╊━╃ ┼ ┼V┼ ┼ ╆━╇━┫
    15┃ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┃   15┃     ┃ ┃ ┃   ┃           ┃   ┃
    16┠┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄W┄┨   16┠ ┼ ┼ ╂ ╂ ╊━╅ ╊━╈━╈━╅ ┼ ╆━╉ W ┨
    17┃ ┆ ┆ ┆ ┆ ┆ ┆ ┆X┆ ┆ ┆ ┆ ┆Y┆ ┆ ┃   17┃     ┃ ┃ ┃ ┃ ┃X┃ ┃ ┃   ┃Y┃   ┃
    18┠┄┼┄Z┄┼┄┼┄┼┄┼┄┼┄┼┄┼0┼┄┼┄┼┄┼┄┼┄┨   18┣━╅ Z ╄━╉ ╂ ╄━╇━╉ ╂0╊━┿━╋━╇━┿━┫
    19┃ ┆ ┆ ┆ ┆ ┆ ┆1┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┃   19┃ ┃     ┃ ┃  1  ┃ ┃ ┃   ┃     ┃
    20┠2┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄3┄┼┄┼┄┼┄┨   20┠2╂ ┼ ┼ ╊━╇━┿━╅ ╂ ╊━╉ 3 ╂ ┼ ┼ ┨
    21┃ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆4┆ ┃   21┃ ┃     ┃     ┃ ┃ ┃ ┃   ┃  4  ┃
    22┠┄┼┄┼┄┼┄┼┄┼┄5┄┼┄┼6┼┄┼┄┼┄┼┄┼┄┼┄┨   22┣━╉ ┼ ╆━╋━╅ 5 ╄━╉6╂ ╊━╈━╉ ┼ ┼ ┨
    23┃ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┃   23┃ ┃   ┃ ┃ ┃     ┃ ┃ ┃ ┃ ┃     ┃
    24┠┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼7┼┄┼┄┼┄┼┄┨   24┠ ╊━┿━╃ ╂ ╊━┿━╈━╉ ╂ ╂7╂ ╄━┿━╈━┫
    25┃ ┆ ┆8┆ ┆ ┆ ┆ ┆ ┆ ┆9┆ ┆ ┆ ┆ ┆!┃   25┃ ┃  8  ┃ ┃   ┃ ┃ ┃9┃ ┃     ┃!┃
    26┠"┼┄┼┄┼┄┼#┼┄$┄┼%┼┄┼┄┼┄┼┄┼┄&┄┼┄┨   26┠"╂ ╆━┿━╉#╂ $ ╂%╂ ╂ ╊━╇━╅ & ╄━┫
    27┃ ┆ ┆ ' ┆ ┆ ┆ ┆ ┆ ┆ ┆ ) ┆ ┆ ┆ ┃   27┃ ┃ ┃ ' ┃ ┃   ┃ ┃ ┃ ┃ ) ┃     ┃
    28┠┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┨   28┠ ╊━╇━┿━╉ ╊━┿━╇━╇━╉ ╊━┿━╇━╈━╅ ┨
    29┃ ┆ ┆)┆ ┆ ┆ ┆ * ┆ ┆ ┆ ┆+┆ ┆,┆ ┃   29┃ ┃  )  ┃ ┃   *   ┃ ┃  +  ┃,┃ ┃
    30┗━┷━┷━┷━┷━┷━┷━┷━┷━┷━┷━┷━┷━┷━┷━┛   30┗━┻━┷━┷━┻━┻━┷━┷━┷━┻━┻━┷━┷━┻━┻━┛

Runtime:  34 sec