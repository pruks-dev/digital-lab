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
# การทดลองที่ N.M <Sub-Experiment>   (H1 for each sub-experiment)
## ขั้นตอนการทดลอง (Procedure)        (H2)
### คำถามท้ายการทดลอง (Questions)    (H3, at end of sub-experiment)
---
# สรุปผลการทดลอง (Conclusion)         (H1, near end)
---
# คำถามท้ายใบงาน (Review Questions)   (H1, final section)
```

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
- Entity names in PascalCase, signal names in lowercase_snake_case
- Port maps with `=>` aligned
- Comments in Thai or English where helpful
- Use `std_logic` and `std_logic_vector` from `ieee.std_logic_1164.all`
- Use `ieee.numeric_std.all` for arithmetic operations

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
- `mux-2to1.svg` — 2-to-1 Multiplexer
- `d-flip-flop.svg` — D Flip-Flop symbol
- `register-4bit.svg` — 4-bit Register block diagram
- `clock-timing.svg` — Clock timing diagram
- `adder-4bit.svg` — 4-bit adder block diagram
- `calculator-system.svg` — Calculator system integration diagram

### Review Questions (คำถามท้ายใบงาน)

- **Must not overlap** with per-experiment questions (คำถามท้ายการทดลอง).
- If a concept is already covered in a per-experiment question, remove the corresponding final review question and renumber.
- Labs already cleaned: 1 (removed 3), 2 (removed 1), 4 (removed 1), 6 (removed 1).

### Lab-Specific Notes

- **Lab 1**: Structure is NOT gate → AND+OR (combined) → build XOR from AND/OR/NOT → Half Adder.
  ICs used: 74HC04 (NOT), 74HC08 (AND), 74HC32 (OR). No 74HC86 (XOR) — students build XOR.
- **Lab 2**: Experiment 2.3 covers NAND Gate as Universal Gate using 74HC00 (added to equipment).
- **Labs 3–8**: FPGA/VHDL labs on DE10-Lite board. No ICs; use Quartus Prime Lite.

### How to Add a New Worksheet

1. Create `docs/lab-N.md` following the H1/H2 structure above
2. Include: Objectives (bullet points, `สามารถ` rules), Equipment, one or more sub-experiments
   (each with procedure, truth tables, and questions), Conclusion (identical text), and Review Questions
3. Reference IC part numbers correctly (e.g., 74HC08, 74HC32, 74HC04, 74HC86, 74HC00)
4. For VHDL-heavy worksheets, include complete code blocks with library declarations
5. Add SVG diagrams to `docs/images/lab-N/` for key circuits
6. Update `mkdocs.yml` `nav` section to include the new worksheet
