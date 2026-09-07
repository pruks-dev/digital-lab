# ใบงานการทดลองที่ 7: เครื่องคิดเลขอย่างง่าย (Simple Calculator)

---

## วัตถุประสงค์

- อธิบายหลักการลบเลขฐานสองด้วยวิธี Two's Complement และการสร้างวงจรบวก/ลบได้
- อธิบายหลักการล็อกผลลัพธ์ด้วย Register และการแสดงผลลัพธ์ติดลบบน 7-Segment Display ได้
- อธิบายความแตกต่างระหว่างวงจรเลขคณิตแบบ combinational กับแบบมีสถานะ (sequential) ได้
- สามารถออกแบบวงจรเครื่องคิดเลขอย่างง่ายด้วย VHDL ได้
- บูรณาการโมดูลจากใบงานก่อนหน้า (Full Adder, Register, bin_to_bcd, bcd_to_7seg) เข้าด้วยกันแบบ Structural ได้

---

## อุปกรณ์ที่ใช้ในการทดลอง

- บอร์ด DE10-Lite จำนวน 1 บอร์ด
- สาย USB Type-A to Mini-B จำนวน 1 เส้น
- คอมพิวเตอร์ จำนวน 1 เครื่อง
- โปรแกรม Quartus Prime Lite Edition
- โปรแกรม USB-Blaster Driver

---

## การทดลองที่ 7.1 เครื่องคิดเลขพื้นฐาน (บวก/ลบ)

ในใบงานที่ 4 นักศึกษาได้สร้าง Ripple Carry Adder 4 บิต (`adder_4bit`) ไว้แล้ว — ใบงานนี้จะนำความรู้ทั้งหมดมาประกอบเป็น **เครื่องคิดเลขอย่างง่าย**:

- ใส่ค่า A (SW3–SW0) และ B (SW7–SW4)
- กด **KEY0** → คำนวณ **A + B** แสดงผลบน HEX1/HEX0
- กด **KEY1** → คำนวณ **A − B** แสดงผลบน HEX1/HEX0
- ผลลัพธ์ **ค้างอยู่บนจอ** จนกว่าจะกดปุ่มใหม่
- ถ้าผลลบติดลบ → HEX5 แสดง "−" และ HEX1/HEX0 แสดงค่าสัมบูรณ์

> **หลักการลบด้วย Two's Complement:** การลบ $A - B$ ทำได้โดยการบวก $A$ กับ **Two's Complement ของ B**:
>
> $$A - B = A + (\overline{B} + 1)$$
>
> นั่นคือ กลับบิตทุกบิตของ B ($\overline{B}$) แล้วบวก 1 — ผลลัพธ์ที่ได้คือคำตอบที่ถูกต้องในระบบ Two's Complement

**ภาพรวมของระบบ:**

![บล็อกไดอะแกรมเครื่องคิดเลขพื้นฐาน](images/lab-7/calculator.svg)

กำหนดให้

- SW3–SW0 แทนค่า A, SW7–SW4 แทนค่า B
- **KEY0** กด = บวก, **KEY1** กด = ลบ
- HEX1/HEX0 แสดงผลลัพธ์ (บวก: 0–30, ลบ: ค่าสัมบูรณ์ 0–15)
- HEX5 แสดง "−" เมื่อผลลบติดลบ
- ผลลัพธ์ค้างอยู่บนจอจนกว่าจะกดปุ่มใหม่

> **ทำไมผลลัพธ์ต้อง "ค้าง"?** — ปุ่ม KEY เป็นแบบ momentary (กดแล้วเด้งกลับ) — ถ้าไม่มี Register ผลลัพธ์จะหายไปทันทีที่ปล่อยปุ่ม — **Register** (จากใบงานที่ 5) ทำหน้าที่ **จำค่า** ไว้ข้าม clock cycle — การจำค่านี้คือ **state** (สถานะ) ของวงจร — เป็นก้าวแรกสู่ FSM ในใบงานที่ 8

### ขั้นตอนการทดลอง

1. สร้างโปรเจกต์ใหม่ชื่อ `lab7_step1` (Top-Level Entity ชื่อ `calculator_top`) — คัดลอกไฟล์ `bcd_to_7seg.vhd` (จากใบงานที่ 4) มาเพิ่มเข้าโปรเจกต์ (**Project → Add/Remove Files in Project**)

2. สร้างไฟล์ `bin_to_bcd.vhd` — **ขยายจากใบงานที่ 4.2** ให้รับอินพุต 5 บิต (0–30) เพราะผลบวก A+B สูงสุด 15+15 = 30 — **คัดลอก Entity ตามตัวอย่าง แล้วเขียน Architecture เอง**:

    ```vhdl
    library ieee;
    use ieee.std_logic_1164.all;

    entity bin_to_bcd is
        port (
            bin  : in  std_logic_vector(4 downto 0);   -- ค่า 0–30
            bcd1 : out std_logic_vector(3 downto 0);   -- หลักสิบ
            bcd0 : out std_logic_vector(3 downto 0)    -- หลักหน่วย
        );
    end entity;

    architecture Dataflow of bin_to_bcd is
    begin
        -- (1) bcd1: with bin select — หลักสิบ: 0 (0–9), 1 (10–19), 2 (20–29), 3 (30)
        --     ใช้ "0011" when others สำหรับค่า 30 (11110)

        -- (2) bcd0: with bin select — หลักหน่วย: bin mod 10
        --     ใช้ "1001" when others สำหรับ 9, 19, 29
    end architecture;
    ```

    > **ขยายจาก 4 บิตเป็น 5 บิต:** ใบงานที่ 4.2 แปลงค่า 0–15 (4 บิต) — ใบงานนี้ผลบวกได้ถึง 30 จึงต้องเพิ่มบิตที่ 5 (`bin(4)`) — หลักสิบมีค่าได้ถึง 3 (เลข 30) — หลักหน่วยแต่ละค่าปรากฏ 3 ครั้ง (เช่น 0 = `00000`, `01010`, `10100`) — ใช้ `|` รวมกรณีที่ให้ค่าเดียวกัน และใช้ `when others` ปิดท้าย

3. สร้างไฟล์ `calculator_top.vhd` — วงจรหลัก: รับ KEY0/KEY1 → คำนวณ + ล็อกผลลัพธ์ → แสดงผล — **คัดลอก Entity ตามตัวอย่าง แล้วเขียน Architecture เอง**:

    แต่ละส่วนของวงจรทำงานดังนี้:

    - **Edge Detect** — ปุ่ม KEY เป็น active-low (กด = `0`) — ต้องจับ **ขอบลง** (การเปลี่ยน `1 → 0`) เพื่อให้กด 1 ครั้ง = 1 เหตุการณ์ — ใช้ Register เก็บค่าปุ่มก่อนหน้า (`key0_prev`, `key1_prev`) แล้วเทียบกับค่าปัจจุบัน
    - **Result Register** — process ที่ทำงานทุก `rising_edge(clk)` — เมื่อมี press ให้คำนวณผลลัพธ์ (บวก/ลบ) แล้วเก็บลง `result` — ถ้าลบติดลบ เก็บค่าสัมบูรณ์และตั้ง `negative = '1'`
    - **bin_to_bcd** — แปลง `result` (5 บิต) เป็น BCD 2 หลัก
    - **bcd_to_7seg ×2** — แปลง BCD แต่ละหลักเป็น 7-segment pattern
    - **HEX5** — แสดง "−" (segment g) เมื่อ `negative = '1'` — นอกนั้นดับ

    ```vhdl
    library ieee;
    use ieee.std_logic_1164.all;
    use ieee.numeric_std.all;

    entity calculator_top is
        port (
            clk  : in  std_logic;                    -- Clock 50 MHz จาก oscillator บนบอร์ด
            a    : in  std_logic_vector(3 downto 0); -- Operand A (SW3–SW0)
            b    : in  std_logic_vector(3 downto 0); -- Operand B (SW7–SW4)
            key0 : in  std_logic;                    -- บวก (KEY0)
            key1 : in  std_logic;                    -- ลบ (KEY1)
            hex5 : out std_logic_vector(7 downto 0); -- เครื่องหมายลบ (HEX5)
            hex1 : out std_logic_vector(7 downto 0); -- หลักสิบ (HEX1)
            hex0 : out std_logic_vector(7 downto 0)  -- หลักหน่วย (HEX0)
        );
    end entity;

    architecture Structural of calculator_top is
        component bin_to_bcd is
            port (
                bin  : in  std_logic_vector(4 downto 0);
                bcd1 : out std_logic_vector(3 downto 0);
                bcd0 : out std_logic_vector(3 downto 0)
            );
        end component;

        component bcd_to_7seg is
            port (
                bcd : in  std_logic_vector(3 downto 0);
                seg : out std_logic_vector(7 downto 0)
            );
        end component;

        signal key0_prev : std_logic := '1';
        signal key1_prev : std_logic := '1';
        signal press0    : std_logic;
        signal press1    : std_logic;
        signal result    : signed(5 downto 0) := (others => '0');  -- ผลลัพธ์ signed 6 บิต (−32..31)
        signal negative  : std_logic := '0';                          -- ผลลบติดลบ? (sign bit)
        signal abs_result : unsigned(4 downto 0);                     -- ค่าสัมบูรณ์ (0–30)
        signal bcd1_sig  : std_logic_vector(3 downto 0);
        signal bcd0_sig  : std_logic_vector(3 downto 0);
    begin

        -- (1) Edge Detect: process(clk) เก็บ key0/key1 ลง key0_prev/key1_prev
        --     แล้วสร้าง press0/press1 = '1' เมื่อปุ่มเปลี่ยนจาก 1 → 0

        -- (2) Result Register: process(clk) — ถ้า press0 → บวก, press1 → ลบ
        --     ใช้ signed: result <= signed(resize(a,6)) + signed(resize(b,6)) หรือลบ
        --     negative <= result(5)  -- sign bit (MSB): '1' = ติดลบ
        --     ถ้า result < 0 → abs_result <= unsigned(0 - result(4 downto 0))  -- 2's complement
        --     else: abs_result <= unsigned(result(4 downto 0))

        -- (3) U_BCD: bin_to_bcd — bin => std_logic_vector(abs_result)

        -- (4) U_DEC1/U_DEC0: bcd_to_7seg — bcd => bcd1_sig/bcd0_sig

        -- (5) hex5: "10111111" เมื่อ negative = '1' (segment g ติด) — นอกนั้น "11111111"

    end architecture;
    ```

    > **Result Register คือ state:** `result` เก็บผลลัพธ์ไว้ข้าม clock cycle — เมื่อกดปุ่มคำนวณแล้ว ต่อให้สลับสวิตช์ A/B ผลลัพธ์บนจอก็ยังคงค่าเดิม จนกว่าจะกดปุ่มใหม่
    >
    > **การแสดงผลลบ:** `result` เป็น **signed** — เมื่อ A < B (เช่น 3 − 5) ผลลัพธ์เป็นค่าลบจริง (−2) — `negative` = sign bit (MSB) = '1' — ก่อนแสดงผลต้องทำ **2's complement** (กลับบิต + 1 หรือ `0 − result`) ให้เป็นค่าสัมบูรณ์ (2) — HEX5 แสดง "−" และ HEX1/HEX0 แสดง 2 — รวมเป็น "−2"

4. **Simulate ด้วย Waveform** — ตรวจสอบความถูกต้องของวงจรก่อนลงบอร์ด:
    - **File → New → University Program VWF** → Insert Node `clk`, `a`, `b`, `key0`, `key1` (input) และ `hex5`, `hex1`, `hex0` (output)
    - ตั้งค่า `clk` เป็นสัญญาณ Clock (period 10 ns) — กด `key0` (ขอบลง) ทดสอบการบวก (เช่น 3+2=5, 9+9=18)
    - กด `key1` (ขอบลง) ทดสอบการลบ (เช่น 5−2=3, 3−5=−2 — ตรวจสอบว่า `hex5` แสดง "−")
    - **Simulation → Run Functional Simulation**
    - ตรวจสอบว่า `hex1`/`hex0` ตรงกับผลบวก/ผลลบที่คำนวณได้ และ `hex5` เปลี่ยนเฉพาะเมื่อผลลบติดลบ

5. กำหนด Pin Assignment:
    - `clk` → PIN_P11 (`MAX10_CLK1_50`)
    - `a(3)`–`a(0)` → SW3–SW0
    - `b(3)`–`b(0)` → SW7–SW4
    - `key0` → KEY0, `key1` → KEY1
    - `hex5(7:0)` → ขา HEX5, `hex1(7:0)` → ขา HEX1, `hex0(7:0)` → ขา HEX0

6. **Compile** (**Processing → Start Compilation**) → **Program** ลงบอร์ดด้วย USB-Blaster

7. ทดลองตามตารางที่ 7.1 — ตั้งค่า A, B แล้วกด KEY0/KEY1 บันทึกผล

#### ตารางที่ 7.1 ผลการทดลองเครื่องคิดเลขพื้นฐาน

| A (ฐานสิบ) | B (ฐานสิบ) | กดปุ่ม | การทำงาน | HEX5 | HEX1 | HEX0 | ผลลัพธ์ฐานสิบ |
| ---------- | ---------- | ------ | -------- | ---- | ---- | ---- | ------------- |
| 3          | 2          | KEY0   | บวก      |      |      |      | 5             |
| 5          | 4          | KEY0   | บวก      |      |      |      | 9             |
| 9          | 9          | KEY0   | บวก      |      |      |      | 18            |
| 5          | 2          | KEY1   | ลบ       |      |      |      | 3             |
| 9          | 4          | KEY1   | ลบ       |      |      |      | 5             |
| 3          | 5          | KEY1   | ลบ       |      |      |      | −2            |

> **สังเกต:** หลังกดปุ่มคำนวณแล้ว ลองสลับสวิตช์ A/B — ผลลัพธ์บน HEX ยังคงค่าเดิม (ถูกล็อกไว้) — จนกว่าจะกด KEY0/KEY1 ใหม่ — นี่คือพฤติกรรมของ **state** — วงจร "จำ" ผลลัพธ์ไว้ได้

### คำถามท้ายการทดลองที่ 7.1

1. วงจรเดียวกันสามารถใช้ทั้งบวกและลบได้อย่างไร — อธิบายหลักการ Two's Complement
2. เพราะเหตุใดผลลัพธ์จึงค้างอยู่บนจอแม้ปล่อยปุ่มและสลับสวิตช์ — อธิบายบทบาทของ Register
3. เมื่อผลลบติดลบ วงจรแสดงผลอย่างไร — HEX5 กับ HEX1/HEX0 แสดงค่าอะไรบ้าง

---

## การทดลองที่ 7.2 Accumulator (วงจรบวกสะสม)

ในข้อ 7.1 เครื่องคิดเลขคำนวณผลลัพธ์ครั้งเดียวแล้วล็อกไว้ — ระบบจริงมักต้องการ **สะสมค่าไปเรื่อย ๆ** เช่น เครื่องคิดเงินที่รวมยอดสินค้าทีละรายการ

**Accumulator** คือวงจรที่ **บวก/ลบค่าเข้ากับค่าที่สะสมไว้** ทุกครั้งที่กดปุ่ม:

$$acc = acc \pm b$$

โดย `acc` (accumulator) เป็น Register ที่เก็บค่าสะสม — ทุกครั้งที่กด KEY0 ค่า `acc` จะเปลี่ยนตาม `b` และทิศทางที่เลือกด้วย SW8 — นี่คือ **state ที่สะสมค่า** ต่อยอดจากแนวคิด Register ในข้อ 7.1

> **จาก Counter สู่ Accumulator:** ในใบงานที่ 6 Counter นับเพิ่มทีละ 1 (`count + 1`) — Accumulator ต่างกันตรงที่บวก/ลบเพิ่มทีละ `b` (ค่าใดก็ได้) แทนที่จะเป็น 1 เสมอ — ทั้งคู่คือ Register ที่ "สะสมค่า" ตามจังหวะ clock

**ภาพรวมของระบบ:**

![บล็อกไดอะแกรม Accumulator](images/lab-7/accumulator.svg)

กำหนดให้

- SW7–SW0 แทนค่า B (ค่าที่บวก/ลบสะสม) — 8 บิต (0–255)
- SW8 เลือกทิศทาง: `0` = บวก, `1` = ลบ
- KEY0 เป็น manual clock — กด 1 ครั้ง = สะสม 1 ครั้ง (rising edge ตอนปล่อยปุ่ม)
- KEY1 เป็น reset — กดเมื่อไหร่ `acc` กลับเป็น 0 ทันที (active-low)
- LEDR9 แสดง **overflow** — ติดเมื่อบวกเกิน 255 หรือลบติดลบ (เกินช่วงแสดงผล 0–255)
- HEX3 แสดง "−" เมื่อผลลบติดลบ (negative)
- HEX2/HEX1/HEX0 แสดงค่าสะสม (0–255)

### ขั้นตอนการทดลอง

1. สร้างโปรเจกต์ใหม่ชื่อ `lab7_step2` (Top-Level Entity ชื่อ `accumulator_top`) — คัดลอกไฟล์ `bcd_to_7seg.vhd` (จากใบงานที่ 4) มาเพิ่มเข้าโปรเจกต์ (**Project → Add/Remove Files in Project**)

2. สร้างไฟล์ `accumulator_top.vhd` — วงจรหลัก: รับ KEY0/SW8 → สะสมค่า → แปลง BCD → แสดงผล — **คัดลอก Entity ตามตัวอย่าง แล้วเขียน Architecture เอง**:

    แต่ละส่วนของวงจรทำงานดังนี้:

    - **Accumulator** — process ที่ทำงานทุก `rising_edge(clk)` (KEY0) — บวก/ลบ `b` เข้า `acc` ตาม `add_sub` — `acc` เป็น Register ที่ "จำ" ค่าสะสมไว้ — เมื่อกด KEY1 (`reset`) ให้ `acc` กลับเป็น 0 — เมื่อผลบวกเกิน 255 หรือผลลบติดลบ ให้ตั้ง `overflow = '1'` (แสดงบน LEDR9) — เมื่อผลลบติดลบ ให้ตั้ง `negative = '1'` ด้วย (แสดง "−" บน HEX3)
    - **bin_to_bcd (Double Dabble)** — process ที่แปลง `acc` (8 บิต) เป็น BCD 3 หลัก — หลักการ: เลื่อนบิตทีละ 1 ไปทางซ้าย — ถ้า BCD digit ใดเกิน 4 ให้บวก 3 ก่อนเลื่อน (เพราะการเลื่อนซ้าย = คูณ 2 — การบวก 3 ช่วย "พก" หลัก) — ทำครบ 8 ครั้ง (เท่าจำนวนบิต)
    - **bcd_to_7seg ×3** — แปลง BCD แต่ละหลัก (ร้อย/สิบ/หน่วย) เป็น 7-segment pattern
    - **HEX3** — แสดง "−" (segment g) เมื่อ `negative = '1'` — นอกนั้นดับ

    ```vhdl
    library ieee;
    use ieee.std_logic_1164.all;
    use ieee.numeric_std.all;

    entity accumulator_top is
        port (
            clk     : in  std_logic;                    -- KEY0 (manual clock)
            reset   : in  std_logic;                    -- KEY1 (reset acc = 0)
            b       : in  std_logic_vector(7 downto 0); -- ค่าที่บวก/ลบสะสม (SW7–SW0)
            add_sub : in  std_logic;                    -- SW8: '0' = บวก, '1' = ลบ
            hex3    : out std_logic_vector(7 downto 0); -- เครื่องหมายลบ (HEX3)
            hex2    : out std_logic_vector(7 downto 0); -- หลักร้อย
            hex1    : out std_logic_vector(7 downto 0); -- หลักสิบ
            hex0    : out std_logic_vector(7 downto 0); -- หลักหน่วย
            overflow : out std_logic                    -- LEDR9: บวกเกิน 255 / ลบติดลบ
        );
    end entity;

    architecture Structural of accumulator_top is
        component bcd_to_7seg is
            port (
                bcd : in  std_logic_vector(3 downto 0);
                seg : out std_logic_vector(7 downto 0)
            );
        end component;

        signal acc      : signed(9 downto 0) := (others => '0');  -- ค่าสะสม signed 10 บิต (−512..511)
        signal negative : std_logic := '0';                        -- ผลลบติดลบ? (sign bit)
        signal abs_acc  : unsigned(9 downto 0);                    -- ค่าสัมบูรณ์ (0–511)
        signal bcd2     : std_logic_vector(3 downto 0);             -- หลักร้อย
        signal bcd1     : std_logic_vector(3 downto 0);             -- หลักสิบ
        signal bcd0     : std_logic_vector(3 downto 0);             -- หลักหน่วย
    begin

        -- (1) Accumulator: process(clk) — rising_edge
        --     ถ้า reset = '0' (active-low): acc <= (others => '0'); overflow <= '0'; negative <= '0'
        --     ถ้า add_sub = '0' (บวก): acc <= acc + signed(resize(unsigned(b), 10))
        --     ถ้า add_sub = '1' (ลบ): acc <= acc - signed(resize(unsigned(b), 10))
        --     negative <= acc(9)  -- sign bit (MSB): '1' = ติดลบ
        --     overflow <= '1' เมื่อ acc > 255 หรือ acc < 0 (เกินช่วงแสดงผล 0–255)

        -- (2) abs_acc: ถ้า acc < 0 → abs_acc <= unsigned(0 - acc)  -- 2's complement
        --     else: abs_acc <= unsigned(acc)

        -- (3) bin_to_bcd: process(abs_acc) — variable temp : unsigned(11 downto 0)
        --     วาง abs_acc ที่บิต 0–9 — loop 10 ครั้ง: add-3 ถ้า digit > 4 แล้วเลื่อนซ้าย
        --     ส่งผลลัพธ์: bcd2/bcd1/bcd0 จาก temp(11..8)/(7..4)/(3..0)

        -- (4) U_DEC2/U_DEC1/U_DEC0: bcd_to_7seg — bcd => bcd2/bcd1/bcd0

        -- (5) hex3: "10111111" เมื่อ negative = '1' (segment g ติด) — นอกนั้น "11111111"

    end architecture;
    ```

    > **`acc` เป็น `signed(9 downto 0)`:** ค่าสะสม 10 บิต (−512..511) — เก็บค่าลบได้จริง (ต่างจาก unsigned ที่วนกลับ) — เช่น 96 − 167 = **−71** (เก็บ −71) — เมื่อติดลบ `negative` = '1' และก่อนแสดงผลต้องทำ **2's complement** ให้เป็นค่าสัมบูรณ์ (71) — HEX3 แสดง "−" และ HEX2/1/0 แสดง 71 — รวมเป็น "−71"

    > **Double Dabble (shift-add-3):** วิธีแปลงเลขฐานสองเป็น BCD — เลื่อนบิตทีละ 1 ไปทางซ้าย — ถ้า BCD digit ใดเกิน 4 ให้บวก 3 ก่อนเลื่อน (เพราะการเลื่อนซ้าย = คูณ 2 — การบวก 3 ช่วย "พก" หลัก) — ทำครบ 8 ครั้ง (เท่าจำนวนบิต) จะได้ BCD 3 หลัก

3. **Simulate ด้วย Waveform** — ตรวจสอบความถูกต้องของวงจรก่อนลงบอร์ด:
    - **File → New → University Program VWF** → Insert Node `clk`, `reset`, `b`, `add_sub` (input) และ `hex3`, `hex2`, `hex1`, `hex0`, `overflow` (output)
    - ตั้งค่า `b = 100` (01100100), `add_sub = 0` — กด `clk` (rising edge) 1 ครั้ง — ตรวจสอบ acc = 100, `overflow` = 0, HEX3 ดับ
    - กด `reset` (`0`) — ตรวจสอบ acc = 0
    - ตั้งค่า `b = 200` (11001000), `add_sub = 0` — กด `clk` 1 ครั้ง — ตรวจสอบ acc = 200, `overflow` = 0
    - ตั้งค่า `b = 100` (01100100), `add_sub = 0` — กด `clk` 1 ครั้ง — ตรวจสอบ acc = 300, **`overflow` = 1**, HEX3 ยังดับ (บวกเกิน ไม่ใช่ติดลบ)
    - เปลี่ยน `add_sub = 1` — กด `clk` 1 ครั้ง — ตรวจสอบ acc = 200 (300 − 100), `overflow` = 0, HEX3 ดับ
    - ตั้งค่า `b = 250` (11111010), `add_sub = 1` — กด `clk` 1 ครั้ง — ตรวจสอบ acc = −50, **`overflow` = 1**, **HEX3 แสดง "−"** (ผลลบติดลบ)
    - ตรวจสอบ BCD: เมื่อ acc = 200 → `bcd2` = 2, `bcd1` = 0, `bcd0` = 0

4. กำหนด Pin Assignment:
    - `clk` → KEY0
    - `reset` → KEY1
    - `b(7)`–`b(0)` → SW7–SW0
    - `add_sub` → SW8
    - `overflow` → LEDR9
    - `hex3(7:0)` → ขา HEX3
    - `hex2(7:0)` → ขา HEX2
    - `hex1(7:0)` → ขา HEX1
    - `hex0(7:0)` → ขา HEX0

5. **Compile** → **Program** ลงบอร์ด

6. ทดลองตามตารางที่ 7.2 — **กด KEY1 (reset) ครั้งเดียวตอนเริ่ม** ให้ acc = 0 แล้วทำตามลำดับ: ตั้งค่า B และ SW8 ตามแถว → กด KEY0 **หนึ่งครั้ง** → บันทึก acc หลังกด — ทำต่อเนื่องโดย **ไม่ reset ระหว่างแถว** (acc ไหลต่อจากแถวก่อนหน้า)

#### ตารางที่ 7.2 ผลการทดลอง Accumulator

| ลำดับ | B (SW7–0) | SW8 | acc ก่อนกด | acc หลังกด KEY0 | overflow (LEDR9) | HEX3 |
| ----- | ---------- | --- | ---------- | --------------- | ---------------- | ---- |
| 1 | 127 | 0 (บวก) | 0 |  |  |  |
| 2 | 131 | 0 (บวก) |  |  |  |  |
| 3 | 89 | 1 (ลบ) |  |  |  |  |
| 4 | 73 | 1 (ลบ) |  |  |  |  |
| 5 | 167 | 1 (ลบ) |  |  |  |  |
| 6 | 251 | 0 (บวก) |  |  |  |  |
| 7 | 53 | 1 (ลบ) |  |  |  |  |
| 8 | 113 | 1 (ลบ) |  |  |  |  |
| 9 | 199 | 0 (บวก) |  |  |  |  |
| 10 | 97 | 0 (บวก) |  |  |  |  |

> **สังเกต:** ค่าสะสมไหลต่อเนื่องทีละ B ทุกครั้งที่กด KEY0 (acc ก่อนกดของแถวถัดไป = acc หลังกดของแถวก่อนหน้า — นักศึกษาเติมเอง) — `acc` เป็น **signed** เก็บค่าลบได้จริง — เมื่อลบติดลบ (ลำดับ 5: 96 − 167 = −71) วงจรทำ **2's complement** ให้เป็นค่าสัมบูรณ์ (71) — HEX3 แสดง "−" และ HEX2/1/0 แสดง 71 — รวมเป็น "−71" — เมื่อบวกเกิน 255 (ลำดับ 2, 10) หรือติดลบ (ลำดับ 5) → **LEDR9 ติด** (overflow) — ต่างกันตรงที่ LEDR9 ติดทั้งบวกเกินและลบติดลบ แต่ HEX3 แสดง "−" เฉพาะลบติดลบ — กด KEY1 เมื่อไหร่ `acc` กลับเป็น 0 ทันที (reset) — นี่คือ **state ที่สะสมค่า** — ต่างจากข้อ 7.1 ที่ state เก็บค่าเดียว (ผลลัพธ์ล่าสุด) ข้อนี้ state สะสมค่าไปเรื่อย ๆ

### คำถามท้ายการทดลองที่ 7.2

1. Accumulator ต่างจาก Counter ในใบงานที่ 6 อย่างไร — และต่างจาก Register ในข้อ 7.1 อย่างไร
2. เมื่อค่าสะสมเกิน 255 หรือลบจนติดลบ เกิดอะไรขึ้นกับ `acc` — `overflow` กับ `negative` ต่างกันอย่างไร — และระบบจริงควรจัดการอย่างไร
3. บทบาทของ SW8 (`add_sub`) คืออะไร — ถ้าไม่มีสัญญาณนี้ วงจรจะทำได้เพียงใด

---

## สรุปผลการทดลอง

อธิบายผลการทดลอง พร้อมวิเคราะห์ความถูกต้องของผลลัพธ์ และอธิบายสาเหตุของข้อผิดพลาด (ถ้ามี)

## คำถามท้ายใบงาน

1. เหตุใดจึงควรออกแบบวงจรเป็นหลายโมดูล (Modular Design) แทนการเขียนเป็นวงจรเดียวขนาดใหญ่
2. หากต้องการเพิ่มการคูณ (×) จะต้องเพิ่มโมดูลใด และอาศัยหลักการใด
3. Result Register (ข้อ 7.1) กับ Accumulator (ข้อ 7.2) — state ทั้งสองแบบต่างกันอย่างไร และแบบใดเหมาะกับระบบที่ต้อง "จำ" ค่า
4. เพราะเหตุใดจึงควรทดสอบหลายชุดข้อมูลก่อนนำวงจรไปใช้งานจริง
5. จากแนวคิด Register (ข้อ 7.1) และ Accumulator (ข้อ 7.2) — จงยกตัวอย่างระบบจริงที่ต้อง "จำค่า" หรือ "สะสมค่า" พร้อมอธิบายว่า state แต่ละค่าหมายถึงอะไร