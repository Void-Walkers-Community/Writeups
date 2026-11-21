# Writeup: d4rkmaze

**Challenge:** solve the maze (`d4rkmaze.png`) to get the flag, in the format `d4rk{...}`.
**Flag:** `d4rk{how_was_the_maze}`

## 1. Recon

The image is 3120×507 px, a very wide maze with START at the bottom left and FINISH at the bottom right. A caption gives the flag format: `d4rk hello world` becomes `d4rk{hello_world}`.

Scanning for dark pixels gave the geometry:

- The outer border is 4 px thick, centred at x=11.5 on the left and x=3107.5 on the right, and at y=11.5 on the top and y=455.5 on the bottom.
- Internal walls fall on a 12 px pitch.
- That gives a grid of **258 columns × 37 rows**.
- The only gaps in the border are in the bottom row, at the first and last cells. These are the entrance and exit.

## 2. Turning the picture into a graph

Cell (r, c) has its centre at `(11.5 + 12c + 6, 11.5 + 12r + 6)`. For each pair of neighbouring cells I sampled a small pixel window on the grid line between them:

- A dark pixel on the vertical line `x = 11.5 + 12(c+1)` means a wall between (r, c) and (r, c+1).
- A dark pixel on the horizontal line `y = 11.5 + 12(r+1)` means a wall between (r, c) and (r+1, c).

I then ran a BFS from the bottom-left cell (36, 0) to the bottom-right cell (36, 257).

```python
def wallv(r,c):  # wall to the right of cell (r,c)
    x=int(11.5+12*(c+1)); y=int(11.5+12*r+6)
    return d[y-1:y+2, x-1:x+2].any()
def wallh(r,c):  # wall below cell (r,c)
    y=int(11.5+12*(r+1)); x=int(11.5+12*c+6)
    return d[y-1:y+2, x-1:x+2].any()
# standard BFS over (r,c), skipping moves blocked by wallv/wallh
```

The BFS found a path of 2236 cells.

## 3. The twist: the path is the message

Overlaying the solution on the maze showed that the route is not a plain walk. It detours into tall, deliberate loops. The decoy branches around it are random, but the solution path hugs a block-letter shape.

Rendering only the path on a blank canvas made the letters clear:

> **D 4 R K  H O W  W A S  T H E  M A Z E**

The 4 is the least clear glyph, but it matches the `d4rk` prefix.

## 4. Flag

Applying the format from the caption (lowercase, words joined by underscores, wrapped in braces) gives:

```
d4rk{how_was_the_maze}
```

---
* [🔙 Back to Miscallaneous Directory](../)
* [🔙 Back to Miscellaneous Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
