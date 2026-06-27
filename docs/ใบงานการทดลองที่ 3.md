---
level: staff
---
# ใบงานการทดลองที่ 3: การใช้งาน FPGA เบื้องต้นด้วย VHDL

---

## วัตถุประสงค์

- อธิบายหลักการทำงานของ FPGA ได้
- สามารถสร้างโปรเจกต์ VHDL ด้วย Quartus Prime Lite ได้
- กำหนดอุปกรณ์เป้าหมาย (Target Device) ได้
- กำหนด Pin Assignment สำหรับ Switch และ LED ได้
- Compile และ Download โปรแกรมลงบนบอร์ด DE10-Lite ได้
- สามารถเขียนวงจรแบบ combinational เบื้องต้นด้วยภาษา VHDL ได้

---

## อุปกรณ์ที่ใช้ในการทดลอง

- บอร์ด DE10-Lite จำนวน 1 บอร์ด
- สาย USB Type-A to Mini-B จำนวน 1 เส้น
- คอมพิวเตอร์ จำนวน 1 เครื่อง
- โปรแกรม Quartus Prime Lite Edition
- โปรแกรม USB-Blaster Driver
- Digital Oscilloscope พร้อม Probes จำนวน 1 ชุด
- Function Generator จำนวน 1 เครื่อง

---

## การทดลองที่ 3.1 การสร้างโปรเจกต์ VHDL

### ขั้นตอนการทดลอง

1. เปิดโปรแกรม Quartus Prime Lite
2. สร้างโปรเจกต์ใหม่
3. กำหนดชื่อโปรเจกต์
4. เลือก Target Device ให้ตรงกับบอร์ด DE10-Lite
5. สร้างไฟล์ VHDL ใหม่
6. บันทึกไฟล์และกำหนดเป็น Top-Level Entity

---

## การทดลองที่ 3.2 การควบคุม LED ด้วย Switch

วงจรที่ต้องการ

```
SW0 -----------> LED0
```

### ขั้นตอนการทดลอง

1. สร้าง Entity สำหรับอินพุตและเอาต์พุต
2. เขียน Architecture แบบ Concurrent Assignment
3. Compile โปรแกรม
4. ตรวจสอบ Error และ Warning
5. กำหนด Pin Assignment
6. Download ลงบอร์ด
7. ทดลองเปิด–ปิด Switch
8. ใช้ Oscilloscope Probe ขา FPGA ที่ต่อกับ LED0 ตาม Pin Assignment — ตรวจสอบระดับแรงดันเอาต์พุตเมื่อ SW0 = 0 และ SW0 = 1
9. วัดแรงดัน V_OH และ V_OL ของ FPGA (มาตรฐาน I/O 3.3V) เปรียบเทียบกับ IC 74HC series ในแลป 1

#### ตารางที่ 3.1 ผลการทดลอง

| SW0 | LED0 |
|-----|------|
|0||
|1||

---

## การทดลองที่ 3.3 การสร้างวงจร AND Gate ด้วย VHDL

วงจรที่ต้องการ

```
SW0 ----\
          AND ------> LED0
SW1 ----/
```

#### ตารางที่ 3.2 Truth Table

| SW0 | SW1 | LED0 |
|-----|-----|------|
|0|0||
|0|1||
|1|0||
|1|1||

### ขั้นตอนการทดลอง

1. เพิ่มอินพุต SW1
2. ใช้ตัวดำเนินการ `and`
3. Compile
4. Download
5. ทดสอบทุกกรณี
6. ใช้ Oscilloscope 2 ช่อง Probe SW0 และ LED0 — สังเกตว่า FPGA ไม่มี Contact Bounce เหมือน Push Button ในแลป 1
7. ใช้ Function Generator ป้อนสัญญาณ Square Wave ความถี่ 1 kHz ที่ SW0 (ผ่าน GPIO) และต่อ SW1 ผ่าน Switch — ดู Waveform เอาต์พุต AND Gate บน Oscilloscope

---

## การทดลองที่ 3.4 การสร้างวงจร Half Adder ด้วย VHDL

ให้ออกแบบวงจร Half Adder โดยใช้

- SW0 = A
- SW1 = B

และกำหนดให้

- LED0 = Sum
- LED1 = Carry

สมการ

```
Sum   = A xor B
Carry = A and B
```

#### ตารางที่ 3.3 Truth Table

| A | B | Sum | Carry |
|---|---|-----|-------|
|0|0|||
|0|1|||
|1|0|||
|1|1|||

### ขั้นตอนการทดลอง

1. เขียนวงจร Half Adder ด้วย VHDL
2. Compile โปรแกรม
3. Download ลงบอร์ด
4. ทดลองครบทั้ง 4 กรณี
5. ใช้ Oscilloscope 2 ช่อง Probe Sum และ Carry จากขา FPGA — เปรียบเทียบ Waveform กับ Half Adder ที่ต่อบน Breadboard ในแลป 1
6. บันทึกผล

#### ตารางที่ 3.4 ผลการทดลอง

| A | B | Sum | Carry |
|---|---|-----|-------|
|0|0|||
|0|1|||
|1|0|||
|1|1|||

---

## สรุปผลการทดลอง

อธิบายผลการทดลอง พร้อมวิเคราะห์ความถูกต้องของผลลัพธ์ และอธิบายสาเหตุของข้อผิดพลาด (ถ้ามี)

## คำถามท้ายใบงาน

1. FPGA แตกต่างจากการใช้ IC Digital Logic อย่างไร
2. การ Compile มีวัตถุประสงค์เพื่ออะไร
3. หาก Pin Assignment ไม่ถูกต้อง จะเกิดผลอย่างไร
4. การเขียน VHDL แบบ Concurrent Assignment เหมาะกับวงจรประเภทใด
5. Half Adder ที่สร้างด้วย VHDL แตกต่างจาก Half Adder ที่ต่อบน Breadboard หรือไม่ เพราะเหตุใด
6. แรงดัน V_OH และ V_OL ของ FPGA (3.3V) แตกต่างจาก IC 74HC series (5V) อย่างไร และมีผลต่อการเชื่อมต่อระหว่างอุปกรณ์หรือไม่
7. การใช้ Oscilloscope Probe FPGA Pin โดยตรงช่วยในการ Debug ด้านใดบ้าง