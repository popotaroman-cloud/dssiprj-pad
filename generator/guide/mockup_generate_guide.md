# คู่มือการสร้าง Standalone Mockup (Mockup Generation Guide)

> ใช้คู่กับ [`fn req/<module>.md`](fn%20req/) (functional requirements) และ `<module>/usecase_description.md` (use case) — ไฟล์นี้บอกวิธี **แปลง FN + Use Case ให้เป็นหน้าจอที่กดใช้งานได้จริง** โดยไม่ต้องมี server/backend/database จริง
>
> ผลลัพธ์ที่ได้ใช้เป็น **สไลด์ 10 · Demo Mockup** ใน [`template_slides.md`](../template/template_slides.md) ได้ทันที (screenshot จาก mockup จริง ดีกว่าภาพนิ่งจาก Figma เพราะพิสูจน์ flow การทำงานได้จริงระหว่างสอบ)

---

## 1. นิยาม Standalone Mockup

**Standalone** = เปิดไฟล์ `.html` ได้ตรงจาก `file://` (ดับเบิลคลิกเปิดในเบราว์เซอร์) หรือรันผ่าน static server ง่ายๆ โดย**ไม่มี**:
- Backend API จริง (Django/Node/ฯลฯ)
- ฐานข้อมูลจริง (PostgreSQL/MySQL)
- Authentication จริง (ระบบ login/session จริง)
- การเรียก network request ไปยัง backend จริง

**แต่ต้องทำได้ครบ:**
- ทุกปุ่ม/ลิงก์ต้อง**กดแล้วมีผลจริง** (เปลี่ยนหน้า, เปลี่ยนสถานะ, แสดงข้อมูลใหม่) — ห้ามมี `href="#"` ที่กดแล้วไม่เกิดอะไรขึ้น
- ทุก use case ใน `usecase_description.md` ต้องมีหน้าจอรองรับและ**เดิน flow ได้ครบตาม Basic Flow / Alternative Flow**
- ข้อมูลที่กรอก/แก้ไขต้อง**คงอยู่** เมื่อเปลี่ยนหน้าไปมา (จำลอง state ของระบบจริงด้วย `localStorage`)

---

## 2. โครงสร้างไฟล์แนะนำ

```
<module>/mockup/
├── index.html          ← จุดเริ่ม: หน้าเลือก role (จำลอง login)
├── mock-data.js         ← ข้อมูลตัวอย่างตาม Data Entities ใน fn req
├── app.js               ← logic การนำทาง + จัดการ state (localStorage)
├── style.css            ← สไตล์ร่วมทั้งหมด
└── screens/             ← 1 ไฟล์ต่อ 1 use case (ตั้งชื่อตามรหัส UC)
    ├── uc01-propose-topic.html
    ├── uc02-assign-advisor.html
    └── ...
```

**ทางเลือกอื่น (แนะนำถ้าอยากพกพาง่ายสุด):** รวมทุกอย่างเป็น **ไฟล์ `.html` เดียว** (CSS/JS inline, ทุกหน้าจอเป็น `<div>` ที่ show/hide ด้วย JavaScript) — เหมาะกับการส่งไฟล์เดียวให้กรรมการเปิดดูเองได้โดยไม่ต้องกังวลเรื่อง path พัง

---

## 3. Data Layer จำลอง (Mock Data + State)

### 3.1 สร้างข้อมูลตัวอย่างจาก Data Entities

ดึง entity ตรงจาก fn req หัวข้อ **"6. Data Entities"** มาเป็น array ของ object ใน `mock-data.js`:

```js
// ตัวอย่างจาก fn req/m9_project_thesis.md — entity: thesis
const mockTheses = [
  { id: 1, student_id: "S001", title_th: "ระบบ...", status: "APPROVED", advisor_id: 12 },
  { id: 2, student_id: "S002", title_th: "การพัฒนา...", status: "DRAFT", advisor_id: null },
];
```

### 3.2 ใช้ localStorage แทนฐานข้อมูลจริง

```js
function loadData(key, fallback) {
  const saved = localStorage.getItem(key);
  return saved ? JSON.parse(saved) : fallback;
}
function saveData(key, data) {
  localStorage.setItem(key, JSON.stringify(data));
}
// โหลดครั้งแรกจาก mock-data.js แล้วเซฟกลับทุกครั้งที่มีการแก้ไข
let theses = loadData("theses", mockTheses);
```

### 3.3 ปุ่ม "รีเซ็ตข้อมูลตัวอย่าง"

ต้องมีปุ่มซ่อนไว้มุมจอ (เช่น footer) เพื่อล้าง `localStorage` กลับสู่ข้อมูลตั้งต้น — จำเป็นเพราะกรรมการอาจกดทดลองจนข้อมูลเพี้ยน ต้องรีเซ็ตก่อนสอบรอบถัดไปได้

---

## 4. Mapping จาก Use Case → หน้าจอ (Traceability)

**กฎ: 1 use case ใน `usecase_description.md` = อย่างน้อย 1 หน้าจอ/1 ส่วนใน mockup**

ทำตารางcheck รายการก่อนเริ่มสร้าง (ป้องกันหลุด):

| UC | ชื่อ Use Case | ไฟล์/section ที่รองรับ | สถานะ |
|----|--------------|------------------------|:---:|
| UC-01 | {{ชื่อ}} | `screens/uc01-....html` | ☐ |
| UC-02 | {{ชื่อ}} | `screens/uc02-....html` | ☐ |
| ... | ... | ... | ☐ |

แต่ละหน้าจอต้องมีองค์ประกอบตรงกับตารางในไฟล์ use case description:

| คอลัมน์ใน usecase_description.md | แปลงเป็นอะไรใน mockup |
|-----------------------------------|------------------------|
| Actor หลัก | เมนู/หน้าจอนี้ต้องแสดงเฉพาะตอน login เป็น role นั้น (ดู §5) |
| Basic Flow (ลำดับเหตุการณ์หลัก) | ลำดับปุ่ม/ฟอร์มบนหน้าจอ ต้องกดตามลำดับได้จริง |
| Alternative Flow | ต้อง trigger error/แจ้งเตือนได้จริงเมื่อทำตามเงื่อนไขนั้น (เช่น กรอกไม่ครบ → แสดง validation message) |
| Post-condition | หลังกดปุ่ม action หลัก ต้องเห็นการเปลี่ยนแปลงจริง (สถานะเปลี่ยน, มีข้อมูลใหม่ในตาราง) |
| กฎทางธุรกิจที่เกี่ยวข้อง (Business Rules) | สูตร/เงื่อนไขต้องคำนวณจริงด้วย JS ตาม §7 (pure logic ห้าม mock) |

---

## 5. Role-based Navigation (จำลอง Login)

**หน้า `index.html`** ให้ทำเป็นหน้าเลือก role (ไม่ใช่ฟอร์ม login จริง):

```html
<h2>เลือกบทบาทเพื่อทดลองใช้งาน</h2>
<button onclick="loginAs('student')">นักศึกษา</button>
<button onclick="loginAs('advisor')">อาจารย์ที่ปรึกษา</button>
<button onclick="loginAs('dept_head')">หัวหน้าภาควิชา</button>
```

```js
function loginAs(role) {
  localStorage.setItem("current_role", role);
  window.location.href = "dashboard.html";
}
```

ทุกหน้าจอต้องเช็ค `current_role` แล้วแสดงเฉพาะเมนู/ปุ่มที่ role นั้นมีสิทธิ์ (ตรงกับตาราง **Actors** ใน fn req) — ห้ามแสดงปุ่มที่ role นั้นไม่มีสิทธิ์ แม้จะเป็น mockup ก็ตาม เพราะเป็นส่วนหนึ่งของการพิสูจน์ว่าออกแบบสิทธิ์ถูกต้อง

**ปุ่ม "สลับบทบาท"** ควรอยู่มุมจอตลอดเวลา (ไม่ใช่ของจริง แต่ช่วย demo ครบทุก role จากไฟล์เดียวโดยไม่ต้อง logout/login ใหม่)

---

## 6. ทำให้ทุกปุ่ม/ลิงก์ทำงานได้จริง

**กฎเหล็ก:** ทุก `<button>`/`<a>` ต้องมี `onclick` หรือ event listener ที่ทำสิ่งใดสิ่งหนึ่งจริง:

| ประเภทปุ่ม | ต้องเกิดอะไรขึ้นจริง |
|-----------|----------------------|
| ปุ่ม "บันทึก/ส่ง" | เขียนข้อมูลใหม่ลง array + `saveData()` + แสดงข้อความยืนยัน + เปลี่ยนหน้า/สถานะ |
| ปุ่ม "อนุมัติ/ปฏิเสธ" | เปลี่ยน `status` field ของ record + refresh หน้าให้เห็นผลทันที |
| ปุ่ม "ค้นหา/กรอง" | filter array จริงด้วย JS แล้ว render ผลใหม่ (ไม่ใช่ตารางนิ่ง) |
| ปุ่ม "ยกเลิก" | เปลี่ยน status เป็น CANCELLED จริง ไม่ใช่แค่กลับหน้าเดิม |
| ลิงก์เมนู | นำทางไปหน้าจริง ต้องไม่ใช่ `href="#"` ค้างที่หน้าเดิม |

**Validation ต้องทำงานจริง** ตาม Alternative Flow เช่น "กรอกไม่ครบ → ระบบแจ้งเตือน" — ต้องเขียน JS ตรวจ field จริงและ `alert()`/แสดงข้อความ error จริง ไม่ใช่แค่ปุ่มกดผ่านตลอด

---

## 7. สิ่งที่ Standalone ทำไม่ได้จริง — และวิธี Simulate แทน

| ฟังก์ชันในระบบจริง | ทำไมทำจริงไม่ได้แบบ standalone | วิธี Simulate |
|---------------------|-------------------------------|----------------|
| **เรียก API ของ backend** (เช่น ดึงรายการข้อมูลจาก server) | ไม่มี network/server จริง | เขียน mock function คืนค่าจาก `mock-data.js` ทันที พร้อม comment `// ระบบจริงเรียก GET /api/...` |
| **Authentication/Login** | ไม่มีระบบ login จริง | ใช้ role switcher ใน §5 แทน ไม่ต้องมี password จริง |
| **อัปโหลดไฟล์** | ไม่มี server เก็บไฟล์จริง | ใช้ `FileReader` API อ่านไฟล์ที่เลือกแล้วแสดง preview ในเบราว์เซอร์ ("อัปโหลดสำเร็จ (จำลอง)") โดยไม่ต้องส่งไปไหนจริง |
| **ส่งอีเมล/แจ้งเตือน** | ไม่มี mail server จริง | แสดง toast/banner "ส่งอีเมลแจ้งเตือนแล้ว (จำลอง)" |
| **สร้างเอกสาร PDF/DOCX** (เช่น รายงานสรุป) | ไม่มี backend generate ไฟล์จริง | ใช้ library ฝั่ง client ที่ทำงานได้แบบ standalone จริง เช่น **jsPDF** (CDN) สร้าง PDF จริงจากข้อมูลใน mockup ได้ทันที หรืออย่างน้อยใช้ `window.print()` เปิด print-preview |
| **ตรวจสอบข้อมูลชนกัน/real-time conflict check** | ไม่มีฐานข้อมูลจริง | เขียนฟังก์ชัน JS ตรวจสอบ overlap จาก array ใน memory ตรงๆ (logic เดียวกัน ผลลัพธ์เหมือนกัน) |
| **คำนวณ/ธุรกิจซับซ้อน** (GPA, weighted score, attainment) | — ทำได้จริง ไม่ต้อง simulate | เขียนสูตรคำนวณจริงด้วย JavaScript ตรงๆ ได้เลย เพราะเป็น pure computation ไม่พึ่ง server |

> **หลักการ:** ฟังก์ชันที่เป็น **pure logic/computation** (คำนวณเกรด, ตรวจ validation, กรอง/ค้นหา) ให้เขียน**จริง**เสมอ อย่า mock เฉยๆ เพราะทำได้ไม่ยากและแสดงความเข้าใจ business logic — ที่ต้อง mock คือเฉพาะส่วนที่ต้องพึ่ง **infrastructure ภายนอก** เท่านั้น (network, storage, auth, mail)

---

## 8. เครื่องมือ/Library แนะนำ (ไม่ต้อง build step)

| งาน | เครื่องมือ | เหตุผล |
|-----|-----------|--------|
| โครงสร้างหน้า | HTML + Vanilla JS | ไม่ต้อง build, เปิด `file://` ได้ตรงๆ |
| Styling เร็ว | Tailwind CSS ผ่าน CDN (`<script src="https://cdn.tailwindcss.com">`) | ไม่ต้อง build, ได้ผลลัพธ์สวยเร็ว |
| Interactivity ที่ซับซ้อนขึ้น (ถ้าต้องการ) | Alpine.js ผ่าน CDN | เบา ไม่ต้อง build, syntax คล้าย Vue |
| สร้าง PDF จริงฝั่ง client | jsPDF ผ่าน CDN | ทำงาน standalone ได้จริง ไม่ต้องมี backend |
| ไอคอน | Lucide/Heroicons (SVG inline หรือ CDN) | เบา ไม่ต้อง build |
| **หลีกเลี่ยง** | React/Vue/Angular ที่ต้อง `npm run build` | ขัดหลัก standalone เว้นแต่จะ bundle เป็น 1 ไฟล์ static จริงจบ |

---

## 9. Checklist ความครบถ้วนก่อนส่งงาน

- [ ] ทุก use case ใน `usecase_description.md` มีหน้าจอรองรับ (เทียบตาราง §4 ครบ ☐ → ✅ ทุกแถว)
- [ ] เปิดไฟล์จาก `file://` ตรงๆ ได้โดยไม่ error (ทดสอบโดยไม่มี internet ยกเว้น CDN ที่ใช้)
- [ ] ไม่มี `href="#"` หรือปุ่มที่กดแล้วไม่เกิดอะไรขึ้นเลยแม้แต่จุดเดียว
- [ ] ข้อมูลที่กรอกคงอยู่หลังเปลี่ยนหน้า (ทดสอบ: กรอกฟอร์ม → ไปหน้าอื่น → กลับมา ต้องเห็นข้อมูลเดิม)
- [ ] สลับ role ได้ และแต่ละ role เห็นเฉพาะเมนู/ปุ่มตามสิทธิ์จริงใน fn req
- [ ] Validation/Alternative Flow ทำงานจริง (ทดสอบ: กรอกผิด/ไม่ครบ ต้องเห็น error จริง)
- [ ] มีปุ่มรีเซ็ตข้อมูลตัวอย่างกลับสู่สถานะเริ่มต้น
- [ ] ฟังก์ชันคำนวณ (ถ้ามีใน module นั้น เช่น GPA, weighted score) คำนวณถูกต้องจริง ไม่ใช่ค่า hardcode
- [ ] จุดที่ระบบจริงต้องเรียก backend มี comment ระบุไว้ (เผื่อกรรมการถามว่า "ของจริงทำงานอย่างไร")
- [ ] ทุก path (CSS/JS/รูป/ลิงก์ระหว่างหน้า) เป็น **relative path** ไม่มี absolute path ขึ้นต้นด้วย `/` เลย (จำเป็นสำหรับ GitHub Pages §11.3)
- [ ] ทดสอบผ่าน local static server (ไม่ใช่แค่ `file://`) อย่างน้อย 1 ครั้งก่อน deploy ขึ้น GitHub Pages

---

## 10. เชื่อมกับสไลด์นำเสนอ

Mockup ที่ทำเสร็จตามคู่มือนี้ใช้ต่อกับ [`template_slides.md`](../template/template_slides.md) **Slide 10 · Demo Mockup** ได้โดยตรง:
- ถ่าย screenshot จากหน้าจอที่ทำงานจริงแทนภาพนิ่งจาก Figma
- หรือ**สาธิตสด** (live demo) แทนสไลด์ในช่วง Slide 10 ได้เลย เพราะไฟล์เปิดได้ทันทีไม่ต้องพึ่ง internet/server ตอนสอบ — ลดความเสี่ยงเรื่อง WiFi ห้องสอบล่ม

---

## 11. Deploy ขึ้น GitHub Pages

Mockup ตามคู่มือนี้เป็น static HTML/CSS/JS ล้วน — ตรงกับสิ่งที่ **GitHub Pages** รองรับพอดี ไม่ต้องแก้โค้ดเพิ่ม

### 11.1 ขั้นตอน
1. Push โฟลเดอร์ mockup ขึ้น GitHub repo (จะ push ทั้ง repo หรือแยก repo เฉพาะ mockup ก็ได้)
2. ไปที่ **Settings → Pages** ของ repo เลือก branch (เช่น `main`) และโฟลเดอร์ (เช่น `/` หรือ `/docs`)
3. ได้ URL รูปแบบ `https://<username>.github.io/<repo-name>/` พร้อมแชร์ให้กรรมการเปิดดูเองได้ทันที

### 11.2 ทำไมควร deploy ขึ้น GitHub Pages แทนเปิด `file://` อย่างเดียว

| ประเด็น | `file://` (เปิดในเครื่อง) | GitHub Pages |
|---------|---------------------------|----------------|
| localStorage | บางเบราว์เซอร์ (โดยเฉพาะ Chrome) จำกัด/บล็อก localStorage ภายใต้ origin แบบ `file://` | ทำงานเสถียรเพราะมี origin จริง (`https://...github.io`) |
| แชร์ให้กรรมการดู | ต้องส่งไฟล์ให้โหลดเอง | ส่งแค่ URL เดียว เปิดจากมือถือ/เครื่องไหนก็ได้ |
| CDN (Tailwind/Alpine/jsPDF) | ทำงานได้ถ้ามี internet ตอนเปิด | เหมือนกัน (ต้องมี internet ทั้งคู่) |

### 11.3 ข้อควรระวัง — ต้องใช้ Relative Path เท่านั้น

GitHub Pages เสิร์ฟจาก **subpath** (`https://<username>.github.io/<repo-name>/...`) ไม่ใช่ root domain — ถ้าใช้ absolute path เช่น `/style.css` หรือ `/screens/uc01.html` จะเปิดไม่เจอ (404) เพราะเบราว์เซอร์จะไปมองหาที่ root ของ `github.io` แทน

- ✅ ใช้: `./style.css`, `screens/uc01.html`, `../mock-data.js`
- ❌ ห้ามใช้: `/style.css`, `/screens/uc01.html`

ทดสอบก่อน push เสมอโดยรันผ่าน local static server (เช่น VS Code Live Server) แทนการเปิด `file://` ตรงๆ เพื่อจับ path ที่พังได้ก่อนขึ้นจริง
