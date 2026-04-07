# Chord Chart Format Spec

## Header
Line 1: Title (Composer surname or brief credit)
Line 2: Feel/style, Key
Then chart begins immediately (no blank line).

## Bars
- 9 chars per bar: 8 chars content + `|`
- `|:` and `:|` replace `|` (2 chars), leaving 7 chars content
- No double barlines `||`
- 4 bars per line standard, adjustable as needed

## Chord Placement
- Single chord: left-aligned, padded to 8 chars
- Two chords: second chord at position 5 (char 5 of 8) by default
  - Move earlier if second chord is long and there is room
  - Move later if first chord is long; minimum 1 space between chords
  - Last chord may abut the barline directly
- Three+ chords: minimum 1 space between each; aim for 9 chars but can exceed

## Repeat Symbol
- `%` centred in the 8-char content field: `   %    |`
- Used for a full bar repeat of the previous bar

## Slash Chords / Bassline Changes
- `Gm7  /F` = Gm7 for first half, Gm7/F second half
- `/Bass` treated as second chord, positioned at midpoint

## Chord Symbols
- Flats: lowercase `b` (e.g. `Bb`, `7b9`, `7b5`)
- Sharps: `#` (e.g. `F#`, `7#9`, `7#5`)
- Major 7: `Δ7`
- Half-diminished: `ø7`
- Diminished: `o` (not degree sign °)
- Augmented dominant: `7#5` (not `+7`)
- Minor: `m` (not `-`)
- No spaces within a chord symbol

## Sections
- Labelled `[A]`, `[B]`, `[Verse]` etc. on their own line
- No blank line between section label and first chord line
- Blank line between sections (after last chord line, before next label)
