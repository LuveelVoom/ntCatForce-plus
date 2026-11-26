# ntCatForce+
Catalyst search utility based on LifeAPI library for outer-totalistic (OT), isotropic non-totalistic (INT), and non-isotropic (MAP) rules. It is derived from [CatForce](https://conwaylife.com/wiki/CatForce), which was originally made for B3/S23.

Usage
--
Run `make`, and then `./CatForce inputfile.in`. Compilation has been tested with `clang` and `g++`, with `clang` being used by default.  `make CC=g++` can be used if you do not have `clang`.

Quick start
--
The easiest way to write an input file is the generator page,
`catforce_in_generator.html`: paste the reaction and the catalysts, and it writes the `.in`
file and warns about the usual mistakes. 

`examples/1.in` is a small complete file:

```
max-gen 100                     # simulate this long
num-catalyst 3                  # exactly 3 catalysts (1-3 would allow 1, 2 or 3)
stable-interval 10              # a catalyst counts as recovered after 10 gens intact
search-area -8 -8 16 16         # where catalysts may be placed (torus centre is 0 0)
pat 2bo$bobo$o3bo$o3bo$o3bo$bobo$2bo! -3 -4      # the active pattern and its offset
filter 24 b2o$obo$bo 5 -3       # goal: this must be present at gen 24
cat 2o$2o 60 0 0 .              # block: may be missing 60 gens; '.' = one orientation
cat 2b2o$3bo$3o$o 10 -2 -2 *    # eater: may be missing 10 gens; '*' = all 8 orientations
output 1.rle
```
`<output>_unique.rle` holds one example of each distinct outcome.

Run `./CatForce file.in --check` first: it prints what will be searched and how many
placements, without searching. Three mistakes give useless output without an error:

- **Orientation character `.`** tries only the orientation pasted. Use `*` unless the catalyst is
  symmetric (see Catalyst Symmetry below).
- **No goal** (`filter`, `match` or `bait`): every reaction where the catalysts recover is reported,
  including the pattern simply dying.
- **Large `max-active`** (how long a catalyst may be missing): slow, and the output fills with junk.
  Start around 10-30.

### Command Line Options

The program can be executed as follows:
 `./CatForce file.in [--check] [--show-input] [--threads N] [--max-time T]
[--max-results N] [--part K/N] [--verbose] [--compile-rule] [--merge F.cfdata ...]`

`--check` parses the file, prints a summary that includes the pattern being searched and the number of catalyst placements, and exits without searching. `--show-input` prints out the input file.

All cores are used by default, but the `--threads N` command line option overrides this.

By default, the program prints 1 status line per second; `--verbose` makes it print a line for every catalyst placement, like standard CatForce.

For the usage of `--compile-rule`, `--part K/N`, and `--merge`, see the "Advanced Command Line Options" section.
`--max-time` and `--max-results` override the corresponding commands in the file.

Input File Format
--
See `examples/p83.in` etc. Some useful lists of catalysts are given in `catlists/`.

Parameters are separated by spaces. Parentheses below
denote optional parameters.

### Table 1: Commands from standard CatForce
| Line | Parameter | Description |
|---|---|---|
| `max-gen` | `n` | Overall maximum generation |
| `start-gen` | `n` | The min gen the _first_ encounter occurs |
| `last-gen` | `n` | The max gen the _first_ encounter occurs |
| `num-catalyst` | `n` or `m-n` | The number of catalysts to place (1 to 5), or a range: `1-3` reports every result with 1, 2 or 3 |
| `num-transparent` | `n` | The number of catalysts marked `transparent` that may appear |
| `group-max` | `name n` | At most `n` searched catalysts from group `name` per result (up to 8 groups; fixed-cats don't count) |
| `group-min` | `name n` | At least `n` searched catalysts from group `name` per result |
| `group-lines` | `name n` | Catalysts from at most `n` different `cat` lines of group `name` (copies of one line count once) |
| `group-zone` | `name rle x y` | Every catalyst of group `name` must lie inside these cells (torus coordinates); repeated lines add up. A catalyst in several groups must satisfy each group's zone, and its own `zone` |
| | | Every group named by a `cat` line needs at least one `group-*` line |
| `stable-interval` | `n` | Gens the catalysts must survive after they catalyse to be considered stable |
| `pat` | `rle` | The active pattern |
| | `(dx dy)` | Offset applied to the active pattern |
| `cat` | `rle` | A catalyst |
| | `max-active` | Number of generations in a row the catalyst may be missing |
| | `dx dy` or `auto` | Where the catalyst RLE's top-left cell goes relative to the catalyst's origin, so negative values put the origin inside the catalyst. `auto` puts it at the middle cell (see "Catalyst coordinates") |
| | `symmetries-char` | Character denoting the symmetry of the catalyst (see below) |
| | `(forbidden rle x y)` | Forbidden pattern around the catalyst (see below) |
| | `(required rle x y)` | Cells that must match the catalyst on its own in every generation (live cells stay ON) |
| | `(antirequired rle x y)` | Cells that must stay OFF in every generation |
| | `(locus rle x y)` | Cells of a catalyst that must interact first |
| | `(mustinclude)` | Solutions must use at least one `mustinclude` catalyst |
| | `(transparent)` | Marked as transparent for the purposes of `num-transparent` |
| | `(check-recovery)` | On its own, the catalyst must be back exactly `max-active` gens after it is placed (see below) |
| | `(sacrificial)` | Does not need to recover |
| | `(period n)` | Periodic catalyst |
| | `(any-phase)` | Accept the catalyst in any phase (any oscillator, not just blinkers). The old name `isblinker` still works |
| | `(max-uses n)` | At most n copies of this catalyst per result |
| | `(group name)` | Member of group `name` for `group-max`; may be repeated for several groups. Ignored on `fixed-cat` |
| | `(zone rle x y)` | This catalyst must lie inside these cells (torus coordinates); may be repeated |
| `symmetry` | `symmetrycode` | Symmetry of the entire pattern |
| `search-area` | `x y w h` | Search area. Forms a union with other `search-area` rectangles |
| `exclude-area` | `x y w h` | Removes a rectangle from the search area; may be combined with other `exclude-area` rectangles |
| `(and/or)filter` | `genOrRange rle dx dy` | Filter that must be matched for the solution to be accepted (see below) |
| `match` | `genOrRange rle` | Filter that checks whether the pattern occurs anywhere |
| `max-junk` | `n` | For filters, the maximum non-matching non-catalyst population |
| `fit-in-width-height` | `w h` | Only allow solutions where all catalysts fit in a `w` by `h` rectangle |
| `also-required` | `rle x y` | Require `rle` to be present in every generation; it is added to the starting pattern |
| `output` | `filename` | Output filename |
| `max-category-size` | `n` | Maximum output row length. A row in the main output holds every catalyst variant with the same outcome; further ones are dropped (keeps the RLE from becoming huge) |
| `full-report` | `filename` | Output filename for solutions ignoring all pattern filters |

---
### Table 2.1: New commands for running the search
| Line | Parameter | Description |
|---|---|---|
| `rule` | `rulestring` | The rule that is being searched. ntCatForce+ supports Life-like, INT, and MAP rules |
| `threads` | `n` | The number of worker threads (default 0 = all cores) |
| `torus` | `64` or `128` | Sets the torus size. Specifying `128` runs `CatForce128`, an auxiliary program built automatically on first use |
| `max-time` | `90`, `15m`, `2h`, `1d` | Stop after this long; results so far are saved |
| `max-results` | `n` | Stop after `n` solutions |
| `verbose` | `yes` | Print a message for every placement of an initial catalyst (like standard CatForce does) |
| `split-level` | `2` | Threads normally take turns on the first catalyst's placements. `2` makes them share the second catalyst's placements too, which helps when there are few first placements for many cores; otherwise it can be much slower (4.5x on a 3-catalyst test) |
| `part` | `K/N` | Run only the K-th of N equal shares of the search (see below) |
| `resume-from` | `n` | A save and resume method that is very buggy. It skips the first n top-level catalyst placements, resuming the search from a specific point |

### Table 2.2: New commands for setting up the search
| Line | Parameter | Description |
|---|---|---|
| `include` | `file` | Insert the lines of another file here (e.g. a catalyst list) |
| `layout` | `file.rle [x y]` | Read a layout file (see "Layout files" section) |
| `legend` | `4=goal:20-max ...` | Change what layout states mean for `layout` lines in this file |
| `bait` | `rle x y [survive n] [absent n]` | A bait object. It is kept if it is perturbed and reappears in the same location afterwards |
| `fixed-cat` | `rle x y max-active [cat options]` | A catalyst already in place (see below) |
| `catalyst-zone` | `rle x y` | Catalysts must lie entirely inside these cells; forms a union with other `catalyst-zone` lines. Used for non-rectangular search areas |
| `no-catalyst-zone` | `rle x y` | No catalyst cell may lie on these cells. Used to subtract a region of the search area |
| `keep-empty` | `rle x y` | These cells must stay dead in every generation; may be combined with other `keep-empty` lines |
| `spaceship-cutoff` | `n` (default 10) or `off` | Don't place catalysts for spaceships more than `n` cells from the rest of the reaction |

### Table 2.3: New commands for filtering results
| Line | Parameter | Description |
|---|---|---|
| `torus-edge` | `flag` (default), `drop`, `off` | Results whose reaction envelope reaches the torus edge: marked (default), dropped, or not checked |
| `keep-duplicates` | `yes` | Disables duplicate removal |
| `keep-interacting` | `yes` | Keeps solutions whose catalysts interact with each other (off by default) |

### Table 2.4: New commands for pruning (speed)
| Line | Parameter | Description |
|---|---|---|
| `lightcone` | `no` | Disables lightcone pruning (derived from [Lightcone](https://github.com/mvr/lightcone), on by default; exhaustive) |
| `lightcone-tight` | `yes` | **Unsafe** (may lose results), deprecated: aim the lightcone pruning only at the mismatched cells |
| `contact-filter` | `yes` | **Unsafe** (may lose results): skip placements where the new catalyst, with no further catalysts, doesn't come back within `max-active` gens |
| `unique-recovery` | `yes` | **Unsafe** (may lose results): skip placements that affect the reaction the same way as an earlier one (see below) |

### Table 2.5: New commands that affect output files
| Line | Parameter | Description |
|---|---|---|
| `sort-by` | `key[,key...]` | The order the output rows are listed in is determined by one of its characteristics: `found` (default), `recovery`, `cells`, `catalysts`, `junk`, `frequency` |
| `output-gap` | `n` | Number of empty cells between solutions in the output RLE (default 10) |
| `unique-output` | `filename` | The output file with one square per distinct outcome, laid out in a square grid. The default is `<output>_unique.rle` |
| `find-oscillators` | `yes`, `max` or `min-max` | Collects catalysed oscillators (periods 3-60 for `yes`) into `<output>_osc.rle` |
| `output-list` | `file` | Writes a text list of every solution, with catalyst RLEs and positions (*Can probably be deprecated*) |

### Explanations of standard CatForce commands and parameters
---
**Catalyst Symmetry**: A character specifying what transformations are
applied to the catalyst:
- `|`: mirror by y.
- `-`: mirror by x.
- `+` or `@`: all 90 degree rotations, for D2- or D2/ invariant catalysts (e.g. boat).
- `x`: for 180 degree symmetrical catalysts.
- `/`: diagonal mirror.
- `*`: all 8 transformations.

**Filters**: Every potential solution will be checked against the
provided filters: only solutions that match the filters will be
kept. There are two variants: `andfilter` and `orfilter`. Every
`andfilter ...` must match and at least one `orfilter ...` must match
for the solution to succeed. A `match` filter pattern-matches the
solution for the specified pattern, and succeeds if the pattern occurs
anywhere (and otherwise behaves like an `andfilter`).

Filters that use a range of generations will only succeed after every
catalyst has interacted.

**Forbidden**: Checks for `rle` in the same location as the
catalyst. If `rle` is matched in any generation, the solution is
dropped. You can see `forbidden` as a "catalyst attached filter",
i.e. the filter location is based on the catalyst location. Intended
to exclude eaters/boat-bits and similar unwanted garbage. You may have
several forbidden patterns per catalyst. See the files in `catlists`
for examples.

**any-phase** (formerly `isblinker`): It denotes a periodic catalyst that is rephased, like the blinker in [54P23](https://conwaylife.com/wiki/54P23). Use it with `period n`; if `period` is omitted, it is identified automatically.

**Catalyst coordinates**: a `cat` line puts the RLE's top-left cell at (`dx`, `dy`). The origin (0, 0) of that frame is the point that `search-area` limits and that output positions refer to. The `x y` of `forbidden`, `required`, `antirequired` and `locus` are in the same frame, so they usually repeat `dx dy`:

    cat 2o$o$b3o$3bo! 10 -2 -2 * required bo$o$b3o$3bo! -2 -2 locus 2o$o! -2 -2

With `auto` in place of `dx dy`, the origin is the middle cell (`dx` = -floor(w/2), `dy` = -floor(h/2)) and the option coordinates count from the RLE's top-left instead. This line is the same as the one above:

    cat 2o$o$b3o$3bo! 10 auto * required bo$o$b3o$3bo! 0 0 locus 2o$o! 0 0

`fixed-cat` options also count from the catalyst's top-left; `zone` is always in torus coordinates. `required` cells must match the catalyst's own evolution without the reaction, so a required dead cell must stay dead. The generator's cell editor shows both frames.

**Search areas**: the allowed catalyst positions are all `search-area`
rectangles combined, minus all `exclude-area` rectangles (with no `search-area`
line the default `-30 -30 60 60` is used). For example, to leave the top-right
quadrant empty:

    search-area -30 -30 60 60
    exclude-area 0 -32 32 32

The area limits where a catalyst's origin (the point set by its `dx dy`) may be
placed, not its whole bounding box. 


**Symmetric searches**: `symmetry s` automatically applies the symmetry `s` to the active
pattern and catalysts. The options for `s` are:
- D2 symmetries: `D2|` (reflect across x = 0), `D2-` (y = 0), `D2\` (y=x) and `D2/` (y=-x)
- C2 symmetries: `C2_1` (bounding box odd by odd), `C2_2` (odd by even), `C2_4` (even by even)
- C4 symmetries: `C4_1` (odd by odd), `C4_4` (even by even)
- D4 symmetries: `D4_+1`, `D4_+2` (odd by even),`D4_+4`, `D4_x1`, and `D4_x4`
- D8 symmetries: `D8_1` and `D8_4`

The options D2 follow [LLS](https://gitlab.com/OscarCunningham/logic-life-search), while the rest of the symmetries follow [Catagolue](https://catagolue.hatsya.com/census). 

There are LLS-inspired alternatives for the other groups:
- D2 symmetries: as above
- C2 symmetries: `C2 C2even C2|even C2-even`
- C4 symmetries: `C4 C4even`
- D4 symmetries: `D4+ D4+even D4+|even D4+-even D4x D4xeven`
- D8 symmetries: `D8 D8even`

The default is "odd" transformations, i.e. those that fix the cell at
(0,0); `verticaleven` and `horizontaleven` (referring to bounding box
dimensions) are recognized as equivalent to `-even` and `|even`,
respectively.

**Mask cache**: the first run for a catalyst list writes a `maskpack3-*` file, which contains collision masks, in the working directory. It is reused by later searches with the same catalysts, rule and torus size, and is safe to delete. Caches in older formats (`maskpack-*`, `maskpack2-*`, `masks/maskraw*`) are ignored.

---
### Explanations of new commands and parameters
---

**Setting the rule**: put a `rule` line anywhere in the `.in` file:

    rule B3/S23                       (Conway's Life - the default if omitted)
    rule B2ei3aeikqr/S2-a3
    rule MAPARYXfhZofugWaH7oaIDogBZofuhogOiAaIDogIAAgAAWaH7oaIDogGiA6ICAAIAAaIDogIAAgACAAIAAAAAAAA 

Accepted formats are Life-like/Hensel strings and Golly
`MAP` strings [9]. Rules containing B0 are not
supported. B3/S23 uses the original fast kernel; every other rule uses lookup tables for evolving the patterns. `--compile-rule` (see below) builds a kernel for your rule that is about as fast as the CGoL one.

**Torus size (`torus 128`)**: changes the torus size from 64x64 to 128x128.
`CatForce` then starts `CatForce128` from the same folder, building it with
`make CatForce128`. Results are the same as on the 64 torus when nothing reaches the edge [8]. It is slower: measured 2.6-3.5x per search on the test files, and 8x with `match` filters.

The center of the torus is `(0,0)`. On a 64x64 torus, the left upper corner is `(-32, -32)` and the lower right corner is `(31,31)`. On a 128x128 torus, these coordinates become `(-64, -64)` and `(63,63)` respectively.

**Torus edge (`torus-edge`)**: by default, results whose reaction envelope wraps around the 64x64 (or 128x128) torus at some point are kept. `torus-edge drop` removes them, and `torus-edge off` skips the reaction-envelope check. 

**Catalyst ranges (`num-catalyst 1-3`)**: Reports every output pattern with a number of catalysts in the given range. It takes only slightly longer than searching for just the larger number of catalysts, since the simpler configurations are visited anyway [5].

**Bait objects**: `bait rle x y [survive n] [absent n]` adds a bait object to the starting pattern. A solution only counts if the bait is *missing* for at least `absent` generations (default 1, use `absent 0` to allow an untouched bait) and is then present again for `survive` consecutive generations (by default, `survive` is equal to `stable-interval`). 

**Fixed catalysts (`fixed-cat`)**: `fixed-cat <rle> <x> <y> <max-active> [options]` puts a catalyst in place before the search starts, and the program still checks if it recovers in the end result. It does not count towards `num-catalyst`. It can be used to add catalysts to an existing partial [7]. 

**Output order (`sort-by`)**: by default, in the output RLE, horizontal rows and the results in each row are written in the order they were found. 

`sort-by` changes the order of the results according to a key. The keys are as follows: `recovery` (catalysts recovered earliest), `cells` (fewest catalyst cells), `catalysts` (fewest catalysts), `junk` (smallest outcome: fewest non-catalyst cells left), `frequency` (rows with the most results first). If there are multiple keys, the program tries them sequentially.

**Escaped spaceships (`spaceship-cutoff`)**: a spaceship that has left the reaction by being more than `n` cells away from the reaction envelope is not catalysed, and stops catalysts from being placed in its path. The default `n` is 10; `spaceship-cutoff off` stops the program from checking for escaped spaceships. 


**Check recovery**: when a catalyst with `check-recovery` is placed, the reaction is run on with no further catalysts; the placement is skipped unless the catalyst is present exactly `max-active` generations later. `contact-filter yes` is a looser, unsafe version applied to every catalyst: it only asks that the catalyst come back at some point within `max-active` generations.

**Unique recovery (`unique-recovery yes`)**: an idea from LocalForce. **Unsafe**: it may lose results. When a catalyst is placed, the rest of the board is followed until that catalyst is back (or `max-gen`); if this history is the same as for an earlier placement from the same partial configuration, the placement is skipped. It does not apply to the last catalyst. It can be combined with `contact-filter`, which is also unsafe: it drops results where a later catalyst rescues an earlier one.

**Searching for oscillators (`find-oscillators`)**: an idea from LocalForce. It checks the pattern for periodicity (between the last catalyst placement and `max-gen`). Oscillators that don't perturb any catalysts are discarded.

Oscillators are stored in`<output>_osc.rle`, but not included in the output of `--merge`. The search takes about 20% more time with this command.

### Miscellaneous changes
---
**Auto-split (part of the infile-generating html page)**: One of the GUI input file generator's components ("auto-split") allows you to paste a pattern in, and it makes an attempt at separating and classifying it to produce the `pattern` part of the infile.

**Catalyst sanity check**: every solution's catalysts (including their mirrored or
rotated images under `symmetry`) are evolved on their own for a full period and
must come back unchanged; otherwise, the solution is rejected. With a global `symmetry`, placements whose mirrored/rotated images would (1)*overlap* or (2)*touch the existing catalysts* are also skipped as soon as they are tried.
`keep-interacting yes` stops the program from skipping the catalyst placements in the 2 aforementioned cases.

Currently, `symmetry` is bugged and flips active regions the wrong way irritatingly often.

---
### Additional info about input files
- unknown commands in the infile produce a warning, and lines starting with `#`, `=` or `-` are comments.
- `#` after a space at the end of a line starts a comment: `num-catalyst 2  # two`.
- `<<` in place of any RLE takes the RLE from the lines after the command. The RLE's header is ignored:

      pat << -3 -2
      x = 3, y = 4, rule = B3/S23
      o$3o$obo$
      2bo!
---
Layout files
---

A "layout file" is a LifeHistory (or any given rule's History version) RLE. It's very similar to a Barrister/Silk input file. The `.rle` file can be run directly:

    ./CatForce search.rle

A standard (`.in`)  input file can also import an RLE with `layout file.rle [x y]`.

**What the states mean** (by default):

| State | Role | Becomes |
|---|---|---|
| 1 (green) | `pat`: the active pattern | one `pat` line |
| 2 (blue) | `zone`: catalysts must lie entirely inside these cells | `catalyst-zone` |
| 3 (white) | `bait`: each object is a bait | `bait` lines |
| 4 (red) | `goal`: each object must appear at this location | `andfilter 1-max ...` lines |
| 5 (yellow)| `fixed`: each object is a catalyst already in place | `fixed-cat ... 20` lines |
| 6 (gray/killer cell)| `keep-empty`: these cells must stay dead in every generation | `keep-empty` |

Every island is read as a separate object.

---
The `.rle` file can also have other commands in addition to the RLE. `#C` lines before the RLE are read as commands, e.g. `#C max-gen 100`. Commands can also be added after the `!`. A `rule` line overrides the RLE's header.

**Legend**: a `legend` line changes the meaning of states by giving them new roles that translate into commands:

    #C legend 4=goal:30-max 5=fixed:15 3=bait:20:2 7=required 8=exclude 9=ignore

Roles: `pat`, `zone`, `exclude` (no catalyst cell may lie here), `bait[:survive[:absent]]`,
`required` (always alive, `also-required`), `fixed[:max-active[:cat options]]`,
`goal[:generations]` (default `1-max`), `keep-empty`, `ignore`. The roles that do not appear by default are `exclude`, `required`, and `ignore`.

States with numerical values up to 24 can be given roles.

`examples/layout_example.rle` is a small complete example, and the generator
page can save the current search as a layout.

### Advanced Command Line Options
---
**Compiled rule kernels (`--compile-rule`)**: for any rule other than B3/S23,
run this once:

    ./CatForce your.in --compile-rule

This writes `rule_kernels.h` with a kernel generated for that rule,
rebuilds the program with `make` (about 10-20 s), and then starts the search. Searches in that rule become about 2x faster [1]. Up to 8 rules are kept, so switching between a few rules needs no rebuild. The shipped `rule_kernels.h`already contains B3/S23-q4i, Cyclotronic and HighLife (B36/S23).

**Splitting a search (`--part K/N`)**: N is the *number of parts* the search is split into. The *K-th part* is being searched. If the program is run with `--part 1/N` through `--part N/N`, the split searches are equivalent to the whole search [3]. It's used to spread a search over several machines, or to complete a long search in pieces. Each part of the search writes its own files, named with a `_partKofN` suffix. 

Ctrl-C stops the search and saves everything found so far. A second Ctrl-C quits immediately without saving, but only if it comes at least 2 s after the first; earlier presses are ignored.

**Combining runs (`--merge`)**: every run also writes `<output>.cfdata`, a plain
text list of its results. These files can be merged:

    ./CatForce <your_infile>.in --merge results_part1of2.cfdata results_part2of2.cfdata
The purpose of this is to combine the output files of fragmented searches that were run with `--part` or `resume-from`.

This removes duplicates across the files, and writes the usual
output files with `_merged` added as a suffix [6]. 

## Checks

1. Measured: a B3/S23-q4i search took 21.2 s before and 8.8 s after, a Cyclotronic search 1.7 s and 1.0 s, with identical results.
2. Measured on a B3/S23-q4i search with stable-interval 15: 2.7x faster at max-gen 30, 1.5x at 45, 1.2x at 60.
3. Checked: the union of the N parts equals a single run, with no overlap.
4. Checked: the set of solutions with several threads equals the single-thread set.
5. Checked: `num-catalyst 1-2` gives exactly the results of `1` plus those of `2`.
6. Checked: with `keep-duplicates yes`, merging 3 parts gives exactly the results of one full run; with duplicate removal the count is the same.
7. Checked: fixing one catalyst of a 2-catalyst search and searching for 1 gives exactly the 2-catalyst results that contain it.
8. Checked on the test suite (`tests/`).
9. Checked against Golly's definition of MAP rules.
