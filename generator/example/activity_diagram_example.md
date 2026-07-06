# ตัวอย่าง — Activity Diagram ที่ทำตามกฎครบทุกข้อ (สถาปัตยกรรม Monolith)

> ตัวอย่างนี้ใช้ **UC-04 · เริ่มจับเวลาโฟกัส (ตีบอส)** จากโปรเจกต์จริง (ดูตาราง use case description ใน `proposal/usecase/usecase_description_2.md`) เพื่อสาธิตกฎทั้งหมดใน [`activity_diagram_generate_guide.md`](../guide/activity_diagram_generate_guide.md)

จุดที่ตัวอย่างนี้สาธิตให้เห็น:
- **ฝั่งซอฟต์แวร์มีเลนเดียวชื่อ "ระบบ"** — ตรวจ pre-condition, คำนวณ HP, สร้างบอส, แจกรางวัล, บันทึกประวัติ ทั้งหมดเกิดในเลนเดียวกัน เพราะเป็น monolith ไม่แยกตาม layer หรือ module (ไม่มีเลน Session Service / Avatar Service / Item Service)
- **ไม่มี Gateway/ตรวจ JWT คั่น** — การล็อกอินเป็น pre-condition ในตาราง use case description ไม่ต้องวาดซ้ำ
- **ไม่มี Audit Log / Notification Service แยก** — "บันทึกเซสชันสำเร็จลงประวัติ" และการแจ้งเตือนช่วงพัก เป็นกิจกรรมปกติในเลนระบบ ไม่ต้อง fork ไป service อื่น
- **Decision ตรงกับ Alternative Flow ในตาราง** — HP = 0 → หมดสติไป UC-30, ยกเลิกกลางทาง → UC-05, โฟกัสไม่ถึง 20 นาที → ไม่ได้ EXP/กล่องสุ่ม
- **Loop ด้วย `repeat`/`repeat while`** — วนแสดงนาฬิกาถอยหลังจนกว่าเวลาครบหรือผู้ใช้กดค้างยกเลิก
- **จุดส่งต่อ use case อื่น (extend) เขียนเป็นกิจกรรม + `stop`** — ไม่ต้องวาด flow ของ UC-05/UC-30 ต่อในไดอะแกรมนี้ เพราะแต่ละ use case มีไดอะแกรมของตัวเอง
- **Monochrome ล้วน** — ไม่มี partition ไหนใช้สีต่างจากที่อื่น

---

```plantuml
@startuml M2_activity_เริ่มจับเวลาโฟกัส
' ══════════════════════════════════════════════════════════════════
' รองรับภาษาไทย — อ่านก่อน render:
' 1. บันทึกไฟล์นี้เป็น UTF-8 (ไม่ต้องมี BOM) เสมอ
' 2. ถ้า render ผ่าน CLI (plantuml.jar) ต้องระบุ flag encoding ด้วย:
'      java -Dfile.encoding=UTF-8 -jar plantuml.jar -charset UTF-8 M2_activity_เริ่มจับเวลาโฟกัส.puml
' 3. บังคับใช้ Smetana (pure-Java layout engine) แทน Graphviz/dot
'    เพราะ Graphviz บางระบบ (โดยเฉพาะ Windows) อ่าน UTF-8 ไม่ตรง ทำให้ตัวอักษรไทยเพี้ยน/หาย
!pragma layout smetana
' 4. ฟอนต์ที่มีสระ/วรรณยุกต์ไทยครบและ render ผ่าน Java ได้เสถียร
'    - Windows: "TH Sarabun New" (ฟอนต์มาตรฐานราชการ) หรือ "Tahoma" (มากับเครื่องอยู่แล้ว)
'    - Linux / PlantUML online server: เปลี่ยนเป็น "Noto Sans Thai" หรือ "Loma"
skinparam defaultFontName "TH Sarabun New"
skinparam defaultFontSize 14
' ══════════════════════════════════════════════════════════════════

!theme plain
skinparam titleFontName "TH Sarabun New"
skinparam titleFontSize 20
skinparam noteFontName "TH Sarabun New"
skinparam ArrowFontName "TH Sarabun New"
skinparam ArrowFontSize 12
skinparam swimlaneTitleFontName "TH Sarabun New"
skinparam swimlaneTitleFontSize 14

' ===== Monochrome เท่านั้น — ไม่มี partition ไหนสีต่างจากที่อื่น =====
skinparam activity {
  BackgroundColor #F5F5F5
  BorderColor #424242
  FontName "TH Sarabun New"
  FontSize 13
  DiamondBackgroundColor #FAFAFA
  DiamondBorderColor #616161
}

title UC-04 เริ่มจับเวลาโฟกัส (ตีบอส) — Activity Diagram

|สมาชิก|
start
:เลือกรายการที่จะโฟกัสและตั้งระยะเวลา\n(หรือใช้ค่าเริ่มต้นจาก UC-24);

|ระบบ|
:ตรวจสอบ pre-condition\n(มีรายการโฟกัส และไม่มีเซสชันอื่นทำงานอยู่);
:คำนวณ HP ล่าสุดของอวตารแบบ lazy evaluation\nจาก last_focus_completed_at;

if (HP ที่คำนวณได้ = 0?) then (ใช่ — หมดสติ)
  :เข้าสถานะ "หมดสติ" ล็อกฟีเจอร์ที่ต้องใช้อวตารแข็งแรง;
  |สมาชิก|
  :ไปทำภารกิจชุบชีวิต (extend UC-30);
  stop
else (ไม่ — ยังไหว)
endif

|ระบบ|
:สร้างบอส HP = เวลาที่ตั้ง × 100;
:เริ่มจับเวลาถอยหลัง;

|สมาชิก|
repeat
  :ดูนาฬิกาถอยหลังคู่กับแถบเลือดบอส;
  |ระบบ|
  :ลดค่า HP บอสตามเวลาจริง (checkpoint ทุก 5 นาที);
repeat while (เวลาครบ หรือ กดค้างยกเลิก (extend UC-05)?) is (ยังไม่ครบ) not (ครบ/ยกเลิก)

if (ยกเลิกกลางทาง?) then (ใช่)
  :ส่งต่อ UC-05 (extend) — คำนวณรางวัลบางส่วน;
  stop
else (ไม่ — จบเซสชันตามเวลา)
endif

if (ระยะเวลาที่โฟกัสจริง ≥ 20 นาที?) then (ใช่)
  :คำนวณกล่องสุ่มตามความยาวเซสชัน (อ้างอิงตาราง UC-10);
  :เพิ่ม EXP ให้อวตาร\n(ถึงเกณฑ์เพิ่มเลเวลอัตโนมัติ);
else (ไม่ — ไม่ได้ EXP/กล่องสุ่ม)
endif

:เติม HP คืนตามสัดส่วน;
:อัพเดต Streak ไฟ;
:บันทึกเซสชันสำเร็จลงประวัติ;
:เริ่มนับเวลาพักอัตโนมัติ;

|สมาชิก|
:เห็นเวลาพักที่เหลือ / รอครบเวลาแล้วเริ่มเซสชันถัดไป\n(กดจบพักก่อนเวลา = extend UC-08);

stop

legend right
  |<back:white>    </back>| Actor ภายนอก (สมาชิก) |
  |<back:#F5F5F5>    </back>| ระบบ (monolith — เลนเดียว) |
endlegend

@enduml
```

---

## เทียบกับตาราง Use Case Description ของ UC-04

| องค์ประกอบในไดอะแกรม | มาจากส่วนไหนของตาราง |
|---|---|
| กิจกรรมแรกของสมาชิก (เลือกรายการ + ตั้งเวลา) | Basic Flow ข้อ 1 |
| ตรวจ pre-condition + คำนวณ HP lazy | Pre-condition + Basic Flow ข้อ 2 |
| กิ่ง HP = 0 → หมดสติ → UC-30 | Alternative/Exception Flow + Business Rules (HP decay) |
| สร้างบอส HP = นาที × 100 | Basic Flow ข้อ 3 + Business Rules (อัตราแปลงเวลา) |
| Loop นับถอยหลัง + checkpoint | Basic Flow ข้อ 4 |
| กิ่งยกเลิกกลางทาง → UC-05 | Alternative Flow + ความสัมพันธ์ `<<extend>>` |
| กิ่ง ≥ 20 นาที → EXP/กล่องสุ่ม | Basic Flow ข้อ 5 + Business Rules (เกณฑ์ 20 นาที) |
| เติม HP / streak / บันทึกประวัติ / เริ่มพัก | Basic Flow ข้อ 5-7 + Post-condition |
| จบพักก่อนเวลา = UC-08 | ความสัมพันธ์ `<<extend>>` |

> **ข้อสังเกต:** ทุก decision และกิ่งทางเลือกในไดอะแกรมต้องชี้กลับไปหาแถวใดแถวหนึ่งในตารางได้เสมอ — ถ้ามีกิ่งที่ตารางไม่ได้พูดถึง แปลว่าตารางตกหล่น หรือไดอะแกรมแต่งเติมเกินสเปก อย่างใดอย่างหนึ่งต้องแก้
