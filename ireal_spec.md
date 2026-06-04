# iReal Pro URL Format — Complete Technical Reference

## URL Schemes

Two formats exist, both importable by current iReal Pro:

- `irealbook://` — plain text, human-readable, produced by pyrealpro library
- `irealb://` — obfuscated format, produced by iReal Pro's own export

### irealb:// Obfuscation Algorithm

The chord block is processed in 50-character chunks. For each full 50-char chunk, swap the first 5 characters with the last 5, and swap characters at positions 10–23 with their mirrors at positions 39–26. Chunks shorter than 50 chars are left plain. The algorithm is its own inverse.

---

## URL Structure

```
irealbook://Song Title=Composer Last First=Style=Key=n=CHORD_BLOCK
```

- Fields delimited by `=`
- Composer is **Last First** order (space separated, no comma)
- The fifth field is always the literal character `n` (legacy placeholder)
- Multiple songs in one URL are separated by `===`
- Style must match iReal Pro's style list (e.g. `Medium Swing`, `Jazz Waltz`, `Bolero`, `Bossa Nova`)
- Key is the root letter with accidental if needed (e.g. `F`, `Bb`, `G`)

---

## Chord Block Structure

The chord block is a flat string of tokens. No newlines. Measures are implicit — defined by barline tokens and cell count.

### Anatomy of a Measure

```
[rehearsal_mark][barline_open][time_sig?][cell],[cell],[cell],[cell][barline_close]
```

Cells are separated by commas. Each cell contains exactly one token (chord, space, or special symbol), optionally prefixed by a size token (`s` or `l`) and optionally suffixed by an alternate chord in parentheses.

---

## The 16-Cell Line Rule

iReal Pro always renders exactly **16 cells per horizontal line**. This is a fixed display constraint independent of time signature. Cell allocation per bar is a layout decision:

| Bars per line | Cells per bar |
|---------------|---------------|
| 4 | 4 |
| 8 | 2 |
| 2 | 8 |

Mixed allocations are valid as long as total cells on the line sum to 16.

**The time signature token only affects the displayed time sig indicator — it does not control cell count.** Always pad measures to the desired visual width regardless of time signature. For standard 4-bars-per-line layout, always use 4 cells per bar (`chord, , , `), even in 3/4, 9/8, etc.

---

## Time Signature Token

Placed at the start of the first measure only:

```
T44  T34  T54  T64  T74  T22  T32  T58  T68  T78  T98  T12
```

---

## Barline Tokens

| Token | Meaning |
|-------|---------|
| `[` | Opening double barline |
| `]` | Closing double barline |
| `{` | Opening repeat barline |
| `}` | Closing repeat barline |
| `\|` | Single barline (standard measure separator) |
| `Z` | Final thick double barline (end of song) |

Barlines are attached directly to the measure string — `barline_open` prepended before the first cell, `barline_close` appended after the last cell (replacing the default `|`).

---

## Rehearsal Marks

Placed **before** the opening barline of the measure they apply to:

| Token | Meaning |
|-------|---------|
| `*A` | Section A |
| `*B` | Section B |
| `*C` | Section C |
| `*D` | Section D |
| `*V` | Verse |
| `*i` | Intro |
| `S` | Segno |
| `Q` | Coda (placed at **end** of measure) |
| `f` | Fermata (placed at **end** of measure) |

---

## Chord Notation

### Root
Standard `A`–`G`. `W` is a special invisible root (see below).

### Quality Suffixes
iReal Pro's chord parser regex:
```
/[A-GW]{1}[+\-\^dhob#suadlt\d]*(\/[A-G][#b]?)?/
```

Only these characters are valid after the root: `+ - ^ d h o b # s u a d l t 0-9`

**Critical:** `m` and `j` are NOT valid suffix characters. This means:
- `Cmaj7` → silently parsed as just `C` (everything after the root dropped)
- `Cm7` → silently parsed as just `C`

Always use iReal's own notation:

| Written | iReal notation |
|---------|---------------|
| `Cmaj7` | `C^7` |
| `Cm7` | `C-7` |
| `Cdim7` | `Co7` |
| `Cm7b5` / `Cø7` | `Ch7` |
| `Caug` | `C+` |
| `Csus4` | `Csus` |
| `C7sus4` | `C7sus` |
| `C7alt` | `C7alt` |
| `Cmaj7#11` | `C^7#11` |
| `C/G` | `C/G` |

### Sharp (`#`) in Chord Names
Must remain as literal `#` in the URL — **not** URL-encoded as `%23`. iReal Pro does not decode `%23` when parsing chord names, causing silent truncation. When URL-encoding, always add `#` to the safe characters:

```python
from urllib.parse import quote
encoded = quote(raw_url, safe=':/=#()')
```

### Enharmonics
iReal Pro supports flat accidentals in chord names (`Bb`, `Eb`, `Ab`, `Db`, `Gb`) but also accepts sharps (`F#`, `C#`, `G#`). Use whatever spelling is musically correct — both work in chord names. Only the **key signature field** has restrictions (must be one of iReal's recognised keys).

### The W Token (Invisible Slash Chord)
`W` renders invisibly, replaced at display time by the root of the previous chord. Used to notate a bass note change without restating the chord quality:

```
F6, ,sW/A,Abo|
```
Renders as: F6 (beats 1–2), F6/A (beat 3), Ab° (beat 4).

---

## Special Cell Tokens

| Token | Meaning |
|-------|---------|
| `x` | Repeat previous measure (simile sign `%`) |
| `p` | Pause / slash (rhythmic slash notehead) |
| `r` | Repeat previous two measures |
| `n` | No chord (N.C.) |
| `W` | Invisible chord root (see above) |
| `XyQ` | Empty beat cell (`irealb://` format only) |

### Simile (`x`) Placement
`x` is a cell token like any other. Place it in beat 2 of empty bars for proper visual spacing:

```
 ,x, ,   ← simile on beat 2, empty beat 1, empty beats 3–4
```

For bars with only 2 or 3 cells, the simile should be placed on beat 1.

```
x, ,   ← simile on beat 1, empty beats 2-3
```


---

## Size Tokens (Chord Width)

Size tokens are **non-counting prefixes** — they attach directly before a chord token with no separator, and do not occupy a cell.

| Token | Meaning |
|-------|---------|
| `s` | Small chord — reduces display width |
| `l` | Large chord — returns to normal display width |

### Rules for s/l Application

A **consecutive run** is a sequence of chord-occupied cells with no empty cell between them and/or the last chord is immediately followed by a barline.

- Apply `s` before the **first chord** of a consecutive run only
- Apply `l` before the **first chord after** a consecutive run (the first chord that has an empty cell following it, or is the sole chord in its bar)
- Chords separated by at least one empty cell do NOT need `s` or `l`

**Examples:**

```
F6, ,C7,       ← gap between F6 and C7 — no s/l needed
sF6,lC7, ,      ← F6 immediately followed by C7 — s on F6 and l on C7
sF6,C7,lAm7,    ← run of three — s on F6 and l on Am7
lG^7, , ,      ← first chord after a run — l on G^7
```

In the `irealb://` obfuscated format, `*m*` is used instead of `s` for small chords.

---

## Alternate (Optional) Chords

Alternate chords appear above the staff in iReal Pro. They are encoded by appending the alternate in parentheses to the cell token they hover over:

```
Bb-7, , (C7), |
```

Here beat 3 is an empty cell with an alternate chord `C7` displayed above it. The parenthesised alternate can also be appended to a chord token:

```
A-7(C7), , , |
```

Add `()` to the URL-encoding safe characters to prevent percent-encoding of parentheses.

---

## Alternate Endings

| Token | Meaning |
|-------|---------|
| `N1` | First ending |
| `N2` | Second ending |
| `N3` | Third ending |
| `N0` | Ending with no number shown |

---

## URL Encoding

```python
from urllib.parse import quote

raw = "irealbook://My Song=Doe John=Medium Swing=C=n=[T44C^7, , , |A-7, , , Z"
encoded = quote(raw, safe=':/=#()')
```

Safe characters that must NOT be percent-encoded:
- `:` — URL scheme separator
- `/` — slash chords (`C/G`) and scheme (`://`)
- `=` — field delimiters
- `#` — chord extensions (`F^7#11`)
- `(` `)` — alternate chord syntax

---

## Complete Measure Reference

```
*A[T44sC^7,lE-7, ,sA-7|lG^7, , , |F^7, ,sW/A,Abo| ,x, , Z
```

Decoded:
1. `*A` — section A rehearsal mark
2. `[` — opening double barline
3. `T44` — 4/4 time signature (first measure only)
4. `sC^7,lE-7, ,sA-7|` — C^7 immediately followed by E-7 (s on C^7 and l on E-7); gap before A-7 then A-7 immediately followed by barline (s on A-7)
5. `lG^7, , , |` — l on G^7 (first chord after a run)
6. `F^7, ,sW/A,lAbo|` — F^7 with gap, then W/A→Abo consecutive run (s on W/A and l on Abo)
7. ` ,x, , Z` — empty beat 1, simile on beat 2, empty beats 3–4, final barline

---

## pyrealpro Library Notes

- Generates `irealbook://` format only
- `TimeSignature(n, d).beats` returns the literal numerator, causing incorrect cell padding for compound meters (e.g. 9/8 → 9 cells instead of 4). **Workaround:** bypass `Measure` for non-standard time signatures and build the chord string directly, using `T98` etc. only as a display prefix on the first measure string.
- `Song.url()` uses `quote(url, safe=':/=')` — **missing `#` and `()`** from safe chars. Always call `url(urlencode=False)` and re-encode manually.
- `s` and `l` are explicitly excluded from beat-count arithmetic in the Measure class.
- Chord strings are passed through verbatim — no validation of chord name syntax.