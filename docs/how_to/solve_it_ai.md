
ctrl + enter = run code in the solveit plataform

# General links

1) [Solve it dashboard](https://solveit.fast.ai/dashboard)
2) [Advent of code puzzles 2023](https://adventofcode.com/2023)


# Polya's how to solve it

https://gist.github.com/jph00/d60301884c56fe063101a7cc6193b3af

# Lesson 1

https://gist.github.com/jph00/99b8fa444f8bb4bd0262419f33eb79ca
https://www.youtube.com/watch?v=4x9wtrDOXac

## Get advent of code data

```python
pip install advent-of-code-data
from aocd import data
```
### Get cookies for authentication 

+ Go to https://adventofcode.com/2023
+ Login
+ Right click over the web page
+ Click Inspect
+ Click over the >> and select the application tab 
+ Go to cookies
+ Search the Value in session 
+ Create the `AOC_SESSION`
```
AOC_SESSION = VALUE from cookies
```

## Puzzle number one

Done

# Lesson 2

https://www.youtube.com/live/X_m7qz5LteY

## Lesson 

Highlight HTML code in Jupyter notebooks

The following code will highlight the words do and dont

```
%%html
<style>
.do {color:green}
.dont {color:red}
</style>
```

# Lesson 4

## Web to md

Some times is useful to convert a web as md for creating documentation or to give a LLM some prior context.

To do so go to a web:

+ Highlight the desired part and then right click on on the web and choose Inspect.

+ Choose copy and then copy outerHTML
+ Paste the copied text into https://web2md.answer.ai/z

## Regex page
 www.regex101.com

## itertools.starmap() for tuples within a list such as [(0,1), (0,1) ]

```python 
from itertools import starmap 
li =[(2, 5), (3, 2), (4, 3)] 
new_li = list(starmap(pow, li)) 
print(new_li)`
```


