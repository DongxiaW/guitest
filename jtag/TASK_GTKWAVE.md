# GUI Agent Benchmark — GTKWave on `jtag.vcd`

A small, end-to-end task to measure how well a GUI agent can drive GTKWave.
Goal: starting from a cold GTKWave launch, produce a readable JTAG-FSM trace
and locate a specific state transition.

## Inputs
- VCD file: [jtag.vcd](jtag.vcd)
- Source (for reference only): [jtag.v](jtag.v), [tb.v](tb.v)
- Expected look: [waveform-ascii.png](waveform-ascii.png)

## Background — what these signals are

This trace is a JTAG (IEEE 1149.1) TAP-controller FSM driven by a random `tms`
stream. The FSM has 16 states; the 4-bit code is on `jtagState` and the
human-readable name of the current state is held in `J_state_ascii` (a 112-bit
register packed with ASCII text).

State encoding (from [jtag.v:57-72](jtag/jtag.v#L57-L72)):

| code | name             | meaning                                  |
|------|------------------|------------------------------------------|
| 0    | testLogicReset   | reset state                              |
| 1    | runTest          | idle / run-test                          |
| 2    | selectDR         | choose Data-Register path                |
| 3    | captureDR        | capture parallel data into DR            |
| **4**| **shiftDR**      | **shift DR serially through TDI/TDO**    |
| 5    | exit1DR          | leave shift, decide pause vs update      |
| 6    | pauseDR          | temporarily halt shifting                |
| 7    | exit2DR          | resume shift or update                   |
| 8    | updateDR         | latch shifted value into parallel output |
| 9–15 | selectIR … updateIR | mirror states for Instruction Register |

So **`shiftDR`** = the state in which JTAG is actually moving data bits — the
"useful work" state. In step 6 below you are looking for the first cycle where
`jtagState == 4` (equivalently, where `J_state_ascii` displays the text
`shiftDR`). If you don't see the text, `J_state_ascii` is probably still
displayed as hex/binary — re-do step 3.

## Task

1. **Launch GTKWave and open the file**
   - Open `jtag/jtag.vcd` (via `File → Open New Tab`, or `gtkwave jtag/jtag.vcd`).

2. **Add the following signals, in this order, to the Signals pane**
   From scope `tb`:
   - `treset`
   - `tck`
   - `tms`
   - `jtagState[3:0]`

   From scope `tb.u0`:
   - `J_state_ascii[111:0]`

3. **Format `J_state_ascii` as ASCII**
   - Right-click `J_state_ascii` → `Data Format → ASCII`.
   - Verify the bus now shows readable strings like `testLogicReset`, `runTest`, `shiftDR`, etc.

4. **Format `jtagState` as decimal**
   - Right-click `jtagState` → `Data Format → Decimal`.

5. **Zoom to fit**
   - toolbar `Zoom Fit`.

6. **Locate the first entry into `shiftDR`** (= first time `jtagState == 4`)

   - **Visual scan:** with `J_state_ascii` set to ASCII (step 3), scroll and
     look for the text `shiftDR`. Zoom in if labels are clipped.

   Then place a **named marker** at that time
   (`Markers → Drop Named Marker`, name it `firstShiftDR`) and record the
   timestamp (ns).

7. **Save the session**
   - `File → Write Save File As…` → `jtag/jtag.gtkw` in this directory.

8. **Capture a screenshot**
   - `File → Grab To File…`.
   - Save as `jtag/gtkwave_result.png`.

## Success Criteria

- All 5 signals are present in the Signals pane in the specified order.
- `J_state_ascii` is displayed as readable text (not hex/binary).
- `jtagState` is shown in decimal.
- A named marker `firstShiftDR` exists at the first `shiftDR` entry.
- `jtag.gtkw` exists and reproduces the view when reopened with
  `gtkwave -a jtag/jtag.gtkw jtag/jtag.vcd`.
- `gtkwave_result.png` shows the labeled waveform with the marker visible.

## Scoring (suggested)

| Step | Points |
|------|--------|
| Open file                                  | 1 |
| Correct 5 signals in correct order         | 2 |
| `J_state_ascii` → ASCII                    | 2 |
| `jtagState` → Decimal                      | 1 |
| Zoom-fit performed                         | 1 |
| Correct `firstShiftDR` timestamp + marker  | 5 |
| Saved `.gtkw` reproduces view              | 1 |
| Screenshot captured                        | — (artifact) |
| **Total**                                  | **13** |

Record wall-clock time from launch to screenshot as the secondary metric.
