# Tiny Number Simon

Tiny Number Simon is a digital memory game for Tiny Tapeout. The game displays
an increasingly long sequence of numbers from 1 to 8 on a single 7-segment
display. The player must repeat the sequence using eight number inputs.

## Features

- Numbers 1 through 8
- One additional number after every completed round
- Maximum displayed score of 63 completed rounds
- A different pseudo-random sequence after reset
- Decimal point indicates when player input is expected
- Decimal score display after a mistake
- No saved best score between games

## How to Play

1. Start the clock.
2. Hold reset low for at least one rising clock edge, then release it.
3. Watch the number shown on the 7-segment display.
4. Wait until the digit becomes blank and the decimal point turns on.
5. Press the input that corresponds to the displayed number.
6. Release the input before pressing the next one.
7. After the complete sequence is entered correctly, the game repeats the
   sequence and adds one new number.
8. Continue until a wrong number is entered.

Example:

```text
computer: 4
user:     4

computer: 4, 2
user:     4, 2

computer: 4, 2, 5
user:     4, 2, 5
```

The first round contains one number, the second contains two numbers, and so on.

## Input Rules

Use one number input at a time:

| Input | Number |
| --- | --- |
| `ui_in[0]` | 1 |
| `ui_in[1]` | 2 |
| `ui_in[2]` | 3 |
| `ui_in[3]` | 4 |
| `ui_in[4]` | 5 |
| `ui_in[5]` | 6 |
| `ui_in[6]` | 7 |
| `ui_in[7]` | 8 |

An input must be released before it can be detected again. After the final
correct input, the game waits until the input is released before starting the
next round.

If several inputs are active simultaneously, the lowest-numbered input has
priority.

## Display

The outputs drive a common-cathode 7-segment display:

| Output | Function |
| --- | --- |
| `uo_out[0]` | Segment `a` |
| `uo_out[1]` | Segment `b` |
| `uo_out[2]` | Segment `c` |
| `uo_out[3]` | Segment `d` |
| `uo_out[4]` | Segment `e` |
| `uo_out[5]` | Segment `f` |
| `uo_out[6]` | Segment `g` |
| `uo_out[7]` | Decimal point |

All outputs are active high. Use one suitable current-limiting resistor per
segment.

### Display States

| State | Display |
| --- | --- |
| Computer sequence | Number 1 through 8, decimal point off |
| Waiting for input | Blank digit, decimal point on |
| Checking input | Blank digit, decimal point on |
| Wrong input | `H`, then the score in decimal |

## Failure and Score

The score is the number of fully completed rounds. A mistake during a round does
not add that round to the score.

After a wrong input, the display shows:

```text
H -> blank -> tens digit -> blank -> units digit -> blank
```

Each item lasts one rising clock edge.

For example, after completing seven rounds:

```text
H -> blank -> 0 -> blank -> 7 -> blank
```

After the final blank, the score returns to zero and a new game begins.

## Reset and New Sequences

Reset is active low.

Keep the clock running while reset is held. The amount of time reset remains
pressed normally changes the sequence selected for the next game.

Reset must include at least one rising clock edge. This provides
pseudo-random variation based on human timing; it is not a true hardware random
number source.

## Clock

The game advances on rising clock edges. Use a slow clock or manual clock steps
so that each displayed number can be observed.

The display changes once per clock edge during:

- Sequence playback
- Failure marker and score display

Player input can remain waiting indefinitely until a new input is pressed.

## Bidirectional Pins

The bidirectional pins are not used:

```text
uio_oe  = 0
uio_out = 0
```

## Required Hardware

- Tiny Tapeout board or compatible Wokwi Tiny Tapeout environment
- Eight active-high buttons or switches
- One common-cathode 7-segment display
- Eight suitable current-limiting resistors
- Slow or manually stepped clock
- Active-low reset input

## Using the Wokwi Diagram

The complete circuit is provided in:

```text
diagram.json
```

Import it into a Tiny Tapeout Wokwi project that supports the
`board-tt-block-input` and `board-tt-block-output` components.

Recommended test:

1. Set all number switches to off.
2. Run a slow clock.
3. Hold reset for several rising edges and release it.
4. Confirm that one number appears with the decimal point off.
5. Confirm that input mode shows only the decimal point.
6. Enter and release the correct number.
7. Confirm that the next sequence repeats the previous number and adds one.
8. Enter a wrong number.
9. Confirm the `H`, blank, tens, blank, units, blank sequence.
10. Confirm that a new game starts afterward.

## Limitations

- The maximum represented score is 63.
- No best score is stored between games.
- Reset timing changes the sequence only while the clock is running.
- The game has no internal clock divider.
- Bidirectional pins are unused.
