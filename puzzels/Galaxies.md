# Galaxies

**IDP solver for the Galaxies puzzle.**

Copyright © Dries007 & Laurens VDW 2019

The rules of the puzzle:

1) Every region should have two-way rotational symmetry,
2) should contain exactly one dot which is in its centre, and
3) should contain no lines separating two of its own squares from each other.
4) No black holes.

## How the Python wrapper works

_Magic of the darkest kind,_ otherwise known as hardcoding.

Big thanks to Pycharm and it's column selection mode for making this much less work. 

The Python "wrapper" script creates the IDP file and runs it. The output of IDP is then parsed and virified. The output is appended to the IDP file as a comment, along with the parsed model grids. **Warning: It overwrites the galaxies.idp file every run!**

This wrapper was created because looking at a set of coordinates to verify the puzzle is not fun. Spending 4 hours creating a script to draw the right Unicode box drawing characters on the other hand, that's my idea of a great saturday night.

When it's run an interactive prompt is shown. This prompt can be used to input model output from the IDP-Web IDE thing. The script will parse it and print the output. This was used to quickly test small changes in the IDP by opening the output file in the Web-IDE and re-running it. **Put an empty line in to print the current state (grid).** The input (centers and grid size) _cannot_ be modified via the interpreter. Change the script to edit those variables. Stop it with a control-C.

The parsing is just dumb regex, but it does the job. Every line is parsed as 1 unit, so you cannot spit (wrap) lines. Unknown lines are ignored, this makes it easy to copy-past multiple models in one go.

The main function is at the bottom of the file, select/set the input there.

The script comes with 4 build in example inputs, all taken from [Simon Tatham's Portable Puzzle Collection](https://www.chiark.greenend.org.uk/~sgtatham/puzzles/js/galaxies.html).

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

## Example

Every character in this grid is a center. It uses A-Z first, but will go on to 0-9 and symbols if it runs out. Using A-Z is nice because lowercase is used to indicate the region association if that option is enabled.

Note: The actual input is a coordinate list; this is what the script prints so the input can be verified.

### 5x5

```text
Input
            1
  01234567890
 0┏━┯━┯━┯━┯━┓
 1┃ ┆ A ┆ ┆ ┃
 2┠┄┼┄┼┄┼┄┼┄┨
 3┃ ┆B┆ ┆ ┆ ┃
 4┠┄┼┄┼┄┼┄C┄┨
 5┃ ┆D┆ ┆ ┆ ┃
 6┠┄┼┄┼┄┼┄┼┄┨
 7┃ ┆ ┆ ┆ ┆ ┃
 8┠E┼┄┼F┼┄┼G┨
 9┃ ┆H┆ ┆ ┆ ┃
10┗━┷━┷━┷━┷━┛
Running...
Runtime:  1.4883130779999192
            1
  01234567890
 0┏━┯━┯━┯━┳━┓
 1┃a┆aAa┆a┃c┃
 2┣━┿━┿━╈━╃┄┨
 3┃b┆B┆b┃c┆c┃
 4┣━┿━┿━╉┄C┄┨
 5┃d┆D┆d┃c┆c┃
 6┣━╈━┿━╉┄╆━┫
 7┃e┃f┆f┃c┃g┃
 8┠E╊━╅F╄━╉G┨
 9┃e┃H┃f┆f┃g┃
10┗━┻━┻━┷━┻━┛
Valid: True
```

### 7x7

```text
Input
            11111
  012345678901234
 0┏━┯━┯━┯━┯━┯━┯━┓
 1┃ ┆ ┆ ┆ A ┆ ┆ ┃
 2┠┄┼B┼┄┼┄┼┄┼┄C┄┨
 3┃ ┆ ┆ ┆ ┆ ┆ ┆ ┃
 4┠┄┼┄┼┄┼┄┼D┼┄┼┄┨
 5┃ E ┆ ┆ ┆ ┆ ┆ ┃
 6┠┄┼┄┼┄┼┄┼┄┼┄┼┄┨
 7┃ ┆ ┆ ┆ ┆ ┆ ┆ ┃
 8┠┄F┄┼┄┼┄┼┄┼┄┼┄┨
 9┃ ┆ ┆ G ┆ ┆ ┆H┃
10┠┄┼┄┼┄┼┄┼┄I┄┼┄┨
11┃ ┆ ┆ ┆ ┆ ┆ ┆ ┃
12┠┄┼┄┼┄┼┄┼┄┼┄┼┄┨
13┃ J ┆ ┆ ┆ K ┆ ┃
14┗━┷━┷━┷━┷━┷━┷━┛
Running...
Runtime:  2.5905204200025764
            11111
  012345678901234
 0┏━┯━┯━┳━┯━┳━┯━┓
 1┃b┆b┆b┃aAa┃c┆c┃
 2┠┄┼B┼┄╊━┿━╉┄C┄┨
 3┃b┆b┆b┃d┆d┃c┆c┃
 4┣━┿━╈━╇━╅D╄━╈━┫
 5┃eEe┃g┆g┃d┆d┃h┃
 6┣━┿━╉┄┼┄╄━┿━╉┄┨
 7┃f┆f┃g┆g┆g┆g┃h┃
 8┠┄F┄╂┄┼┄╆━┿━╉┄┨
 9┃f┆f┃gGg┃i┆i┃H┃
10┣━┿━╃┄┼┄╂┄I┄╂┄┨
11┃g┆g┆g┆g┃i┆i┃h┃
12┣━┿━╅┄┼┄╊━┿━╉┄┨
13┃jJj┃g┆g┃kKk┃h┃
14┗━┷━┻━┷━┻━┷━┻━┛
Valid: True
```

### 10x10

This is with region associations and inner walls not printed, it's too busy in such a large grid.

```text
Input
            11111111112
  012345678901234567890
 0┏━┯━┯━┯━┯━┯━┯━┯━┯━┯━┓
 1┃ A    B     C      ┃
 2┠ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┼D┨
 3┃    E              ┃
 4┠ ┼ ┼ ┼ ┼ ┼F┼ G ┼ ┼ ┨
 5┃     H          I  ┃
 6┠ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┨
 7┃J                  ┃
 8┠ ┼ ┼K┼ ┼ ┼ L ┼ ┼ ┼M┨
 9┃                   ┃
10┠N┼ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┨
11┃  O         P   Q  ┃
12┠ ┼ ┼R┼ ┼ ┼ ┼ ┼ ┼ ┼ ┨
13┃      S      T     ┃
14┠ U ┼ ┼ ┼ V ┼ ┼ ┼ ┼ ┨
15┃     W             ┃
16┠ ┼ ┼ ┼ ┼ ┼ ┼ ┼ ┼ X ┨
17┃            Y      ┃
18┠ Z ┼ ┼0┼ ┼ ┼ ┼ ┼ ┼ ┨
19┃    1         2    ┃
20┗━┷━┷━┷━┷━┷━┷━┷━┷━┷━┛
Running...
Runtime:  8.016023898999265
            11111111112
  012345678901234567890
 0┏━┯━┳━┯━┯━┳━┯━┯━┳━┳━┓
 1┃ A ┃  B  ┃  C  ┃ ┃ ┃
 2┣━┿━╇━┿━┿━╋━╈━┿━╉ ╂D┨
 3┃    E    ┃ ┃   ┃ ┃ ┃
 4┣━┿━╈━┿━╈━╉F╂ G ╂ ╊━┫
 5┃   ┃ H ┃ ┃ ┃   ┃I┃ ┃
 6┣━╅ ╄━┿━╉ ╄━╇━┿━╉ ╂ ┨
 7┃J┃     ┃       ┃ ┃ ┃
 8┣━╉ ┼K┼ ╂ ┼ L ┼ ╂ ╂M┨
 9┃ ┃     ┃       ┃ ┃ ┃
10┠N╊━╈━╅ ╄━╈━╈━╅ ╊━╉ ┨
11┃ ┃O┃ ┃   ┃ ┃P┃ ┃Q┃ ┃
12┣━╇━╉R╊━╈━╃ ╊━╇━╋━╇━┫
13┃   ┃ ┃S┃   ┃ T ┃   ┃
14┠ U ╊━╇━╉ V ╊━┿━╉ ┼ ┨
15┃   ┃ W ┃   ┃   ┃   ┃
16┣━┿━╋━┿━╉ ╆━╃ ┼ ╂ X ┨
17┃   ┃   ┃ ┃  Y  ┃   ┃
18┠ Z ╊━╅0╄━╉ ┼ ╆━╉ ┼ ┨
19┃   ┃1┃   ┃   ┃2┃   ┃
20┗━┷━┻━┻━┷━┻━┷━┻━┻━┷━┛
Valid: True
```

### 15x15

This is with region associations and inner walls not printed, it's too busy in such a large grid.

```text
Input
            111111111122222222223
  0123456789012345678901234567890
 0┏━┯━┯━┯━┯━┯━┯━┯━┯━┯━┯━┯━┯━┯━┯━┓
 1┃ ┆A┆ ┆ ┆B┆ ┆ ┆ ┆ ┆ C ┆ ┆ ┆ ┆ ┃
 2┠┄┼┄┼┄D┄┼┄┼E┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┨
 3┃ ┆ ┆ ┆ ┆ ┆ ┆ F ┆ G ┆ ┆ ┆H┆ ┆ ┃
 4┠┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┨
 5┃ ┆ ┆ I ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┃
 6┠┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┨
 7┃ ┆ ┆J┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┃
 8┠┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄K┄┼┄┼┄┼┄┨
 9┃L┆ ┆ ┆ ┆ ┆ ┆M┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┃
10┠┄┼┄N┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┨
11┃ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆O┆ ┆ ┆ ┃
12┠┄┼┄┼┄┼P┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼Q┨
13┃R┆ ┆ ┆ ┆ ┆ ┆ ┆S┆ ┆ ┆ ┆ ┆ ┆ ┆ ┃
14┠┄┼┄┼┄┼┄┼T┼┄U┄┼┄┼┄┼┄┼V┼┄┼┄┼┄┼┄┨
15┃ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┃
16┠┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄W┄┨
17┃ ┆ ┆ ┆ ┆ ┆ ┆ ┆X┆ ┆ ┆ ┆ ┆Y┆ ┆ ┃
18┠┄┼┄Z┄┼┄┼┄┼┄┼┄┼┄┼┄┼0┼┄┼┄┼┄┼┄┼┄┨
19┃ ┆ ┆ ┆ ┆ ┆ ┆1┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┃
20┠2┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄3┄┼┄┼┄┼┄┨
21┃ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆4┆ ┃
22┠┄┼┄┼┄┼┄┼┄┼┄5┄┼┄┼6┼┄┼┄┼┄┼┄┼┄┼┄┨
23┃ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┆ ┃
24┠┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼7┼┄┼┄┼┄┼┄┨
25┃ ┆ ┆8┆ ┆ ┆ ┆ ┆ ┆ ┆9┆ ┆ ┆ ┆ ┆!┃
26┠"┼┄┼┄┼┄┼#┼┄$┄┼%┼┄┼┄┼┄┼┄┼┄&┄┼┄┨
27┃ ┆ ┆ ' ┆ ┆ ┆ ┆ ┆ ┆ ┆ ) ┆ ┆ ┆ ┃
28┠┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┼┄┨
29┃ ┆ ┆)┆ ┆ ┆ ┆ * ┆ ┆ ┆ ┆+┆ ┆,┆ ┃
30┗━┷━┷━┷━┷━┷━┷━┷━┷━┷━┷━┷━┷━┷━┷━┛
Running...
Runtime:  34.13562569500209
            111111111122222222223
  0123456789012345678901234567890
 0┏━┳━┳━┯━┳━┳━┳━┯━┳━┯━┯━┯━┳━┯━┯━┓
 1┃i┃A┃d┆d┃B┃e┃f┆f┃c┆cCc┆c┃h┆h┆h┃
 2┠┄╄━╉┄D┄╊━╉E╂┄┼┄╊━┿━╈━┿━╃┄┼┄┼┄┨
 3┃i┆i┃d┆d┃i┃e┃fFf┃gGg┃h┆h┆H┆h┆h┃
 4┠┄┼┄╄━┿━╃┄╄━╉┄┼┄╊━┿━╉┄┼┄┼┄╆━┿━┫
 5┃i┆i┆iIi┆i┆i┃f┆f┃k┆k┃h┆h┆h┃k┆k┃
 6┣━╅┄╆━╈━╅┄┼┄╊━╈━╃┄┼┄╄━┿━┿━╃┄┼┄┨
 7┃l┃i┃J┃p┃i┆i┃m┃k┆k┆k┆k┆k┆k┆k┆k┃
 8┠┄╊━╇━╉┄╊━╅┄╂┄╂┄┼┄┼┄┼┄K┄┼┄┼┄┼┄┨
 9┃L┃n┆n┃p┃t┃i┃M┃k┆k┆k┆k┆k┆k┆k┆k┃
10┠┄╂┄N┄╂┄╂┄╊━╉┄╂┄┼┄╆━┿━╈━╅┄┼┄╆━┫
11┃l┃n┆n┃p┃t┃u┃m┃k┆k┃v┆v┃O┃k┆k┃q┃
12┣━╋━┿━╉P╂┄╂┄╄━╋━╈━╃┄┼┄╄━╇━┿━╉Q┨
13┃R┃z┆z┃p┃t┃u┆u┃S┃v┆v┆v┆v┆v┆v┃q┃
14┣━╃┄┼┄╂┄╂T╂┄U┄╊━╃┄┼┄┼V┼┄┼┄╆━╇━┫
15┃z┆z┆z┃p┃t┃u┆u┃v┆v┆v┆v┆v┆v┃w┆w┃
16┠┄┼┄┼┄╂┄╂┄╊━╅┄╊━╈━╈━╅┄┼┄╆━╉┄W┄┨
17┃z┆z┆z┃p┃t┃1┃u┃X┃6┃0┃v┆v┃Y┃w┆w┃
18┣━╅┄Z┄╄━╉┄╂┄╄━╇━╉┄╂0╊━┿━╋━╇━┿━┫
19┃2┃z┆z┆z┃t┃1┆1┆1┃6┃0┃3┆3┃4┆4┆4┃
20┠2╂┄┼┄┼┄╊━╇━┿━╅┄╂┄╊━╉┄3┄╂┄┼┄┼┄┨
21┃2┃z┆z┆z┃5┆5┆5┃1┃6┃9┃3┆3┃4┆4┆4┃
22┣━╉┄┼┄╆━╋━╅┄5┄╄━╉6╂┄╊━╈━╉┄┼┄┼┄┨
23┃"┃z┆z┃8┃#┃5┆5┆5┃6┃9┃7┃&┃4┆4┆4┃
24┠┄╊━┿━╃┄╂┄╊━┿━╈━╉┄╂┄╂7╂┄╄━┿━╈━┫
25┃"┃8┆8┆8┃#┃$┆$┃%┃6┃9┃7┃&┆&┆&┃!┃
26┠"╂┄╆━┿━╉#╂┄$┄╂%╂┄╂┄╊━╇━╅┄&┄╄━┫
27┃"┃8┃'''┃#┃$┆$┃%┃6┃9┃)))┃&┆&┆&┃
28┠┄╊━╇━┿━╉┄╊━┿━╇━╇━╉┄╊━┿━╇━╈━╅┄┨
29┃"┃)┆)┆)┃#┃*┆***┆*┃9┃+┆+┆+┃,┃&┃
30┗━┻━┷━┷━┻━┻━┷━┷━┷━┻━┻━┷━┷━┻━┻━┛
Valid: True
```
