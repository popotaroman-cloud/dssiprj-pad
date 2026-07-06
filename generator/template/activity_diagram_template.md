# Template — Activity Diagram แบบมี Swimlane (`.puml`) — สถาปัตยกรรม Monolith

> วิธีใช้: คัดลอกโค้ด PlantUML ด้านล่างไปที่ `<module>/activity_<ชื่อฟังก์ชัน>.puml`
> แทนที่ `{{...}}` ด้วยข้อมูลจริง และลบ/เพิ่ม activity, decision, loop ตามจำนวนจริงของกระบวนการ
> ต้องทำตามกฎใน [`activity_diagram_generate_guide.md`](../guide/activity_diagram_generate_guide.md) ทุกข้อ โดยเฉพาะ **ฝั่งซอฟต์แวร์มีเลนเดียวชื่อ "ระบบ"**, **ไม่มี Gateway/Audit Log/cross-cutting**, และ **ห้ามใช้สีหลายโทน (monochrome เท่านั้น)**
> ดูตัวอย่างที่กรอกครบแล้วได้ที่ [`activity_diagram_example.md`](../example/activity_diagram_example.md)

---

```plantuml
@startuml {{module}}_activity_{{ชื่อฟังก์ชัน}}
' ══════════════════════════════════════════════════════════════════
' รองรับภาษาไทย — อ่านก่อน render:
' 1. บันทึกไฟล์นี้เป็น UTF-8 (ไม่ต้องมี BOM) เสมอ
' 2. ถ้า render ผ่าน CLI (plantuml.jar) ต้องระบุ flag encoding ด้วย:
'      java -Dfile.encoding=UTF-8 -jar plantuml.jar -charset UTF-8 {{module}}_activity_{{ชื่อฟังก์ชัน}}.puml
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

' ===== Monochrome เท่านั้น — ห้ามให้แต่ละ partition มีสีต่างกัน =====
skinparam activity {
  BackgroundColor #F5F5F5
  BorderColor #424242
  FontName "TH Sarabun New"
  FontSize 13
  DiamondBackgroundColor #FAFAFA
  DiamondBorderColor #616161
}

title {{ชื่อฟังก์ชัน}} — Activity Diagram

' ===== Actor ภายนอก (human) ที่ริเริ่มกระบวนการ =====
|{{Actor}}|
start
:{{กิจกรรมเริ่มต้นของ actor}};

' ===== ฝั่งซอฟต์แวร์มีเลนเดียวชื่อ "ระบบ" — ห้ามแยกตาม layer/module =====
|ระบบ|
:{{ตรวจสอบ pre-condition / เงื่อนไขเริ่มต้น}};

' ===== ตัวอย่าง decision — ลบออกถ้าฟังก์ชันนี้ไม่มีเงื่อนไขแตกกิ่ง =====
if ({{คำถามเงื่อนไข}}?) then ({{กรณีไม่ผ่าน}})
  :{{แจ้งเตือน/จัดการกรณีไม่ผ่าน}};
  stop
else ({{กรณีผ่าน}})
endif

:{{กิจกรรมหลักของระบบ}};

' ===== ตัวอย่าง loop/retry — ลบออกถ้าฟังก์ชันนี้ไม่มีการวนซ้ำ =====
|{{Actor}}|
repeat
  :{{กิจกรรมที่ต้องทำซ้ำ}};
  |ระบบ|
  :{{ตรวจสอบเงื่อนไข}};
repeat while ({{คำถามเงื่อนไข}}?) is ({{ยังไม่ผ่าน}}) not ({{ผ่านแล้ว}})

' ===== บันทึกผล + แจ้งผู้ใช้ — เป็นกิจกรรมปกติในเลนระบบ ไม่ต้อง fork ไป audit log =====
:{{บันทึกข้อมูลลงฐานข้อมูล}};
:{{แสดงผล/แจ้งเตือนผู้ใช้}};

stop

legend right
  |<back:white>    </back>| Actor ภายนอก (human) |
  |<back:#F5F5F5>    </back>| ระบบ (monolith — เลนเดียว) |
endlegend

@enduml
```
