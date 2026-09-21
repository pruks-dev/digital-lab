# AGENTS.md - Digital Lab Repository

## Project Overview

MkDocs site for a Thai-language digital logic lab course
(`lab-1.md` through `lab-8.md`). The worksheets cover:

- Basic logic gates (AND, OR, XOR), Half Adder, Full Adder, Multiplexers
- FPGA basics with VHDL on the DE10-Lite board (Quartus Prime Lite)
- Circuit แบบ combinational และ sequential, counters, arithmetic circuits
- System integration of a calculator on FPGA

Source files are in `docs/`, rendered with **mkdocs-material**.

---

## Commands

### Build / Serve

```bash
# Install dependencies
pip install -r requirements.txt

# Serve locally with live reload
mkdocs serve

# Build static site to site/
mkdocs build
```

### Adding VHDL Code

If you add VHDL source files (`.vhd`), place them in `docs/src/`.
There is no automated compilation pipeline — students compile manually in Quartus Prime Lite.

---

## Code Style Guidelines

### Markdown Files

- **Language**: Thai (primary), with technical English terms for IC names, VHDL keywords,
  signal names, and component references
- **File naming**: `lab-N.md` pattern (N = 1-8)
- **Encoding**: UTF-8
- **Line endings**: LF (Unix-style)

### Markdown Structure (consistent across all worksheets)

```
# ใบงานการทดลองที่ N: <Title>     (H1, top of file)
---
## วัตถุประสงค์ (Objectives)         (H2)
---
## อุปกรณ์ที่ใช้ในการทดลอง (Equipment)  (H2)
---
## การทดลองที่ N.M <Sub-Experiment>   (H2 for each sub-experiment)
### ขั้นตอนการทดลอง (Procedure)        (H3)
#### ตารางที่ N.X <Table>             (H4 — per-table headings)
### คำถามท้ายการทดลอง (Questions)    (H3, at end of sub-experiment)
---
## สรุปผลการทดลอง (Conclusion)         (H2, near end)
---
## คำถามท้ายใบงาน (Review Questions)   (H2, final section)
```

> **Header hierarchy follows Lab 2 as canonical reference.** Sub-experiments use `##` (H2); tables use `####` (H4); procedure and questions use `###` (H3). When a sub-experiment contains sub-parts (e.g. 3.3.1–3.3.3), the sub-parts use `###` (H3) and their internal headers step down to `####` (H4).

### Markdown Formatting Rules

- Use `---` horizontal rules between major sections
- Use `##` for section headers under experiments, `###` for sub-sections
- Tables use GFM pipe syntax with header separators (`|---|---|`)
- Align table columns with dashes: `| --- | --- | --- | --- |`
- Use `1.` for ordered lists (Markdown auto-numbers)
- Use `-` for unordered lists
- Bold (`**text**`) for emphasis on key terms
- No HTML, no inline CSS, no footnotes

### Truth Table / Result Table Convention

```markdown
| Signal_A | Signal_B | Output |
| -------- | -------- | ------ |
| 0        | 0        |        |
```

- Input columns first, output columns last (blank for students to fill)
- **No "คาดหวัง" or "ทดลอง" column labels** — students record actual results only
- **No pre-filled answer values** in output columns
- First row: always all-zeros
- Use 0/1 binary values only
- For decimal test data tables (e.g. Lab 7), use the same blank-output convention

### Timing Diagram Recording Convention

For **time-dependent behavior** (latch gating, edge-triggering, register sampling),
record results with a timing diagram template instead of a truth table:

```markdown
#### Timing Diagram ที่ N.X <Description>   (H4 — same slot as tables)

![alt text](images/lab-N/timing-<circuit>.svg)
```

- Template SVG: input waveforms pre-drawn as solid lines with SW/KEY pin annotations;
  output rows show the signal name only — **students draw the waveform themselves**
- No pre-drawn output values (same blank-output rule as truth tables)
- Stimulus sequence must match the simulation/board procedure step-by-step
- Add a `> **วิธีบันทึก:**` note explaining what students must draw and observe
- Use plain truth tables when the experiment measures discrete logic states
  (e.g., forbidden-state exploration), not time behavior

### Objectives Format

- Use **bullet points** (`-`), not numbered lists
- Omit the prefix `เมื่อสิ้นสุดการทดลอง นักศึกษาจะสามารถ`
- Use `สามารถ` **only** for creation/design verbs: สร้าง, ออกแบบ, เขียน, แสดงผล
- Skip `สามารถ` for comprehension verbs: อธิบาย, อ่าน, ประกอบ, ตรวจสอบ, วิเคราะห์, กำหนด, ทดสอบ, บูรณาการ, ประยุกต์ใช้
- End each bullet with `ได้`

### Conclusion Format

Identical text across all 8 labs:

```
อธิบายผลการทดลอง พร้อมวิเคราะห์ความถูกต้องของผลลัพธ์ และอธิบายสาเหตุของข้อผิดพลาด (ถ้ามี)
```

- No dotted answer lines (`.....`)
- No horizontal rule (`---`) after the text
- No blank spaces for writing

### VHDL Code Blocks (when embedded in Markdown)

- Use fenced code blocks with `vhdl` language tag
- **Always include full code**: `library`, `use`, `entity`, `architecture` — not just snippets
- Entity names in PascalCase, signal names in lowercase_snake_case
- Architecture names follow style: `Structural` (gate-level), `Dataflow` (concurrent assignment), `Behavioral` (process)
- Port maps with `=>` aligned
- Comments in Thai or English where helpful
- Use `std_logic` and `std_logic_vector` from `ieee.std_logic_1164.all`
- Use `ieee.numeric_std.all` for arithmetic operations

### VHDL Architecture Style Guide

| Style | Description | Used When | Example |
|-------|------------|-----------|---------|
| **Structural** | Instantiate gate-level components | Teaching gate↔code mapping | `s_not <= not s; y <= and_s0 or and_s1;` |
| **Dataflow** | Concurrent signal assignment (`when-else`, `with-select`) | Combinational circuits | `y <= a when s = '0' else b;` |
| **Behavioral** | Sequential via `process(clk)` | Flip-Flop, Register, State Machine | `process(clk) begin if rising_edge(clk) then...` |

> **Lab 3–4**: Use Structural + Dataflow only. **Lab 5+**: Introduce Behavioral with `process` and `rising_edge`.
> When introducing `std_logic_vector` for the first time, show both individual-signal and vector approaches side-by-side as a comparison pattern.

### `std_logic_vector`: `to` vs `downto`

```vhdl
signal a : std_logic_vector(0 to 9);      -- MSB = a(0), LSB = a(9)  ❌ not standard
signal b : std_logic_vector(9 downto 0);  -- MSB = b(9), LSB = b(0)  ✅ standard
```

> **Always use `downto`** — it matches binary number convention where bit 0 = LSB. Teach this when vector is first introduced (Lab 3.2).

### Example VHDL style

```vhdl
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

entity entity_name is
    port (
        clk   : in  std_logic;
        reset : in  std_logic;
        input_a : in  std_logic_vector(3 downto 0);
        result  : out std_logic_vector(7 downto 0)
    );
end entity;

architecture Behavioral of entity_name is
begin
    -- concurrent statements or process
end architecture;
```

### Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Markdown files | `lab-N.md` | `lab-3.md` |
| VHDL entities | PascalCase | `HalfAdder`, `FullAdder` |
| VHDL signals | lowercase_snake_case | `input_a`, `carry_out` |
| VHDL constants | UPPER_SNAKE_CASE | `CLOCK_FREQ` |
| Truth table signals | PascalCase or A, B, Y | `Sum`, `Carry`, `A`, `B` |

### Error Handling

- Markdown: N/A (no runtime)
- VHDL within documents: use assertions in testbenches only
- No exception handling; VHDL uses `assert` for verification
- Board-level troubleshooting in Thai in the worksheet text

### Git / Version Control

- `.gitignore` should exclude `site/`, `.DS_Store`, `__pycache__/`
- No CI/CD configured
- Commit messages in Thai or English, describing the worksheet change

### SVG Diagrams

SVG diagrams are stored in `docs/images/` and embedded via standard Markdown:
```markdown
![alt text](images/lab-N/filename.svg)
```

Existing diagrams:
- `ic-74hc04.svg`, `ic-74hc08.svg`, `ic-74hc32.svg`, `ic-74hc86.svg`, `ic-74hc00.svg` — IC pinouts
- `gate-symbols.svg` — Logic gate symbols with truth tables (supports fragment `#not`, `#nand`, `#and`, `#or`, `#xor`)
- `nand-gate.svg` — NAND gate symbol (used in Lab 2)
- `breadboard.svg` — Breadboard structure
- `xor-from-gates.svg` — XOR from AND/OR/NOT
- `half-adder.svg` — Half Adder block diagram
- `full-adder.svg` — Full Adder block diagram (redrawn; HA1/HA2 in same row, Cin routes through gap)
- `mux-2to1.svg` — 2-to-1 Multiplexer block diagram
- `mux-from-gates.svg` — MUX 2-to-1 internal gate-level circuit (NOT+AND×2+OR)
- `d-flip-flop.svg` — D Flip-Flop symbol
- `register-4bit.svg` — 4-bit Register block diagram
- `clock-timing.svg` — Clock timing diagram
- `adder-4bit.svg` — 4-bit adder block diagram
- `calculator-system.svg` — Calculator system integration diagram
- `lab-6/clock-divider.svg` — Clock Divider with Binary Counter block diagram
- `lab-6/ripple-counter.gif` — Ripple Counter animation (replaces old SVG)
- `lab-6/rtc-chain.svg` — Real-Time Clock enable chain (Mod-10/Mod-6 cascade)
- `lab-6/state-position.svg` — State Position: 6-digit window on 8-char message
- `lab-6/scrolling-message.svg` — Scrolling Message system (6 lanes: ROM + char_to_7seg per HEX)
- `lab-6/timing-ripple-counter.svg`, `lab-6/timing-binary-counter.svg` — timing templates (6.1, 6.2)
- `mini-project/vga-system.svg` — FSM + VGA system block diagram (inputs SW/KEY · GPIO · ADC → FSM → draw logic → VGA controller → monitor)
- `mini-project/vga-timing.svg` — VGA 640×480@60 horizontal timing (visible/FP/sync/BP regions, HS/VS waveforms)
- `mini-project/stack-cpu.svg` — (unused, kept) Simple Stack CPU system block diagram (Control + Datapath)
- `mini-project/stack-operation.svg` — (unused, kept) Stack operations: PUSH/ADD/SUB (wrap+flag)/MUL (HI:LO) + rules box
- `mini-project/fsm-cpu-state.svg` — (unused, kept) CPU FSM state diagram (FETCH/EXECUTE/HALT)

### Review Questions (คำถามท้ายใบงาน)

- **Must not overlap** with per-experiment questions (คำถามท้ายการทดลอง).
- If a concept is already covered in a per-experiment question, remove the corresponding final review question and renumber.
- Labs already cleaned: 1 (removed 3), 2 (removed 1), 3 (removed 1), 4 (removed 1), 6 (removed 1).

### Lab-Specific Notes

- **Lab 1**: Structure is NOT gate → AND+OR (combined) → build XOR from AND/OR/NOT → Half Adder.
  ICs used: 74HC04 (NOT), 74HC08 (AND), 74HC32 (OR). No 74HC86 (XOR) — students build XOR.
- **Lab 2**: Experiment 2.3 covers NAND Gate as Universal Gate using 74HC00 (added to equipment).
- **Lab 3**: FPGA introduction — covers all basic DE10-Lite I/O (SW0–SW9, LED0–LED9, HEX0 7-segment).
  MUX progression: gate-level (Structural) → VHDL (Dataflow) → 4-bit scaling → MUX + 7-segment application.
  Introduces `std_logic_vector` with individual-vs-vector comparison and `to` vs `downto`.
- **Lab 4**: Combinational circuit design from Truth Table to 7-Segment display. 3 experiments:
  4.1 BCD to 7-Segment (Structural — K-map → gate equations → Schematic/VHDL),
  4.2 Binary to Dual 7-Segment (Dataflow for bin_to_bcd + Structural component reuse),
  4.3 Full Adder + Ripple Carry (Structural, component reuse ×4).
  Architecture: Structural + Dataflow only — **no Behavioral** (no `process`, no sequential).
  K-map: SOP (group `1`) or POS (group `0`) — students choose. SOP/POS must be used correctly.
  Each experiment: Schematic **or** VHDL (not both). Simulation waveform (.vwf) before board test.
  Component reuse emphasized — `bcd_to_7seg` from 4.1 reused in 4.2 and 4.3.
  No Behavioral VHDL — this lab is purely combinational. No adder/subtractor — moved to later lab.
  Questions: 2 per experiment, 3 end-of-lab (Structural comparison, component reuse, Schematic vs VHDL).
- **Lab 5**: Sequential circuits — RS Latch (table recording, includes forbidden state) →
  RS/D Gated Latches, D Flip-Flops (Master-Slave + Behavioral), Register 4-bit
  (**timing-diagram recording** via `images/lab-5/timing-*.svg` templates).
  KEY0 as manual clock — Active-Low: rising edge occurs on button release.
  Behavioral (`process` + `rising_edge`) introduced here; same stimulus for 5.2.2 and 5.2.3
  so students compare Structural vs Behavioral waveforms.
- **Lab 6**: Counter & Clock — Ripple Counter (Structural, student writes architecture) →
  Clock Divider (GENERIC) → Behavioral Binary Counter → Real-Time Clock (HH:MM:SS, Mod-N
  GENERIC reuse + enable chain) → Stopwatch (run flag + edge detect, no debounce — DE10
  KEY has hardware debounce) → Scrolling Message (state position).
  **Timing-diagram recording** for 6.1 (Ripple Counter) and 6.2 (Binary Counter) via
  `images/lab-6/timing-*.svg` templates — emphasizes ripple (staggered) vs synchronous
  (all bits change together) behavior.
  Recording: 6.3 RTC = checklist + board photo; 6.4 Stopwatch = table; 6.5 Scrolling = table;
  6.3.1 (message_rom) = simulate waveform only, no table.
  Questions: 1–2 per experiment (6.1: 1, 6.2: 2, 6.3: 2).
  `hours_counter` BCD split must slice `(3 downto 0)` (5-bit `hour` → 4-bit `units`).
  No Oscilloscope/Function Generator — FPGA board only.
- **Labs 3–8**: FPGA/VHDL labs on DE10-Lite board. No ICs; use Quartus Prime Lite.
  **No Oscilloscope or Function Generator references** — these labs use only the FPGA board (Switch, LED, 7-Segment).
  Equipment list should omit: Digital Oscilloscope, Function Generator, IC part numbers.
- **Lab 8**: FSM Multiplier (Shift-and-Add) — **anti-AI-spoiler policy**: student-written code
  (e.g. `multiplier_fsm`, `multiplier_top` architectures) uses **entity-only skeletons** —
  code blocks provide library + entity only; architecture contains a Thai comment placeholder
  like `-- นักศึกษาเขียน Architecture เอง`. No internal signal declarations, no implemented
  branches, no enumerated state types, no numbered VHDL-equivalent hints (state table keeps
  assignment symbols as spec).   Blockquotes may describe concepts and name `numeric_std`
  functions but must not translate table rows into VHDL statements.
- **Mini Project**: Open-ended FSM + VGA project (`mini-project.md`, nav entry "Mini Project (FSM + VGA)") —
  groups of 2, students choose their own topic (no proposal step, no topic approval). Mandatory
  minimum: **FSM ≥ 3 states** (state diagram + state table in report) and **VGA output 640×480 @ 60 Hz**
  (on-screen content not prescribed — students define it in their own spec). Inputs free choice:
  SW/KEY, GPIO (×2 headers, 3.3 V only, shared GND), ADC **inside MAX 10** (on-die 12-bit SAR,
  6 channels **ADC_IN0–ADC_IN5** via Arduino Shield header JP8, 0–5 V; Analog Front-End halves
  5 V → 2.5 V, so V_input = value/4095 × 5.0; access only via **Altera Modular ADC IP** from
  IP Catalog — no hand-written SPI; no onboard potentiometer — test with external pot/sensor
  through breadboard + shared GND; ref. DE10-Lite manual 3.6–3.7 and 5.6 ADC Measurement). VGA principle section is detailed enough to implement:
  scan concept as dual Mod-N counters (ties to Lab 6), 640×480@60 timing table (visible/porch/sync,
  800×525, pixel clock 25.175 MHz, hsync 96 px, vsync 2 lines, active-low), pixel clock options
  (÷2 → 25 MHz / PLL / 50 MHz + enable), VGA signals incl. VGA_BLANK_N/VGA_SYNC_N/VGA_CLK,
  drawing = position comparison + color MUX, FSM updates object-position registers.
  Example idea list (Pong, ADC voltmeter/bar-graph, memory game, scoreboard, traffic light,
  slot machine, reaction game, etch-a-sketch, animation) — name + 1–2 sentence description, no how-to.
  External demo-video links in the idea section (inspiration only, not how-to): VHDLwhiz Pong on
  DE10-Lite clips `youtube.com/watch?v=w4JPbEQHHTA` and `youtube.com/watch?v=U-9T54B-v3Y`,
  plus `youtu.be/k_7IV0U2JhQ` (VCL lab, video game on DE10-Lite).
  **Anti-AI-spoiler (stronger than Lab 8)**: principle-description document only — **no VHDL
  code blocks at all** (no skeletons, no entity templates); concepts via text, tables, SVG
  diagrams and blockquotes only. **Structure is a project brief, NOT a worksheet** — no
  Objectives, no Equipment, no step-by-step procedures, no blank recording tables, no skills-map
  table. Sections: ภาพรวม → ข้อกำหนดหลัก (Requirements) → อินพุตที่มีบนบอร์ด → หลักการ VGA →
  ตัวอย่างแนวคิดโปรเจกต์ → สเปกโปรเจกต์ที่ต้องกำหนดเอง (5-item spec requirement: goal, FSM spec,
  I/O, VGA screen, testing) → เกณฑ์ความสำเร็จ (Acceptance Criteria checklist) → ข้อกำหนดการส่งงาน
  (3 items: Quartus project, report — spec 5 ข้อ goes inside the report, member roles — demo)
  + rubric NOT public — kept in `private/mini-project-rubric.md` (outside docs/, never published
  to the site; mini-project.md only states group 30 + individual 10 = 40 points and that details
  are announced in class): group 30 (system 12 = FSM 5/VGA 5/inputs 2 · VHDL 7 = module split
  3/style 2/reuse 2 · report 8 = spec 2/diagram-table 3/waveform-photos 2/issues+roles 1 ·
  topic difficulty 3 as 0–3 level, easy topics not penalized) + individual 10/person (explain
  own part 5 / whole-system 3 / short scenario 2 — members may score differently).
  Old Stack CPU SVGs kept but unused: `mini-project/stack-cpu.svg`, `stack-operation.svg`,
  `fsm-cpu-state.svg`.

### How to Add a New Worksheet

1. Create `docs/lab-N.md` following the H1/H2 structure above
2. Include: Objectives (bullet points, `สามารถ` rules), Equipment, one or more sub-experiments
   (each with procedure, truth tables, and questions), Conclusion (identical text), and Review Questions
3. Reference IC part numbers correctly (e.g., 74HC08, 74HC32, 74HC04, 74HC86, 74HC00)
4. For VHDL-heavy worksheets, include complete code blocks with library declarations
5. Add SVG diagrams to `docs/images/lab-N/` for key circuits
6. Update `mkdocs.yml` `nav` section to include the new worksheet
