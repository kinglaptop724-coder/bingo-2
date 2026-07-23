# Math Bingo (เวอร์ชันออนไลน์ — ไม่ต้องพึ่ง Google Apps Script)

เกมนี้แก้จากเดิมที่ใช้ Google Apps Script (`Code.gs` + `google.script.run`) มาใช้
**Firebase Realtime Database** แทน เพื่อให้ฝากไฟล์ไว้บน GitHub Pages แล้วเล่นออนไลน์ได้เลย
โดยไม่ต้องมีเซิร์ฟเวอร์ของตัวเอง

ไฟล์ที่ใช้จริงมีแค่ 2 ไฟล์:
- `index.html` → หน้านักเรียน
- `admin.html` → หน้าคุณครู (ควบคุมเกม)

(ไม่ต้องใช้ `Code.gs` อีกต่อไป เพราะ GitHub Pages เป็น static hosting รันสคริปต์ฝั่งเซิร์ฟเวอร์ไม่ได้)

---

## ขั้นตอนที่ 1: สร้าง Firebase Project (ฟรี)

1. ไปที่ https://console.firebase.google.com/ แล้วล็อกอินด้วย Google
2. กด **Add project** ตั้งชื่อโปรเจกต์ตามใจ (เช่น `math-bingo`) → กด Continue ไปเรื่อยๆ จนสร้างเสร็จ
3. ในเมนูซ้าย ไปที่ **Build > Realtime Database** → กด **Create Database**
   - เลือก location ที่ใกล้ (เช่น Singapore/asia-southeast1)
   - เลือกโหมด **Start in test mode** (ใช้งานได้เลยชั่วคราว ไม่ต้องล็อกอิน — เหมาะกับการทดสอบในห้องเรียน)
4. เมื่อสร้างเสร็จ จะเห็น URL ของฐานข้อมูล เช่น
   `https://math-bingo-xxxxx-default-rtdb.asia-southeast1.firebasedatabase.app`
   → เก็บ URL นี้ไว้

## ขั้นตอนที่ 2: เอา Firebase Config มาใส่ในโค้ด

1. ในหน้า Firebase Console ไปที่ **Project settings** (รูปเฟือง มุมบนซ้าย)
2. เลื่อนลงมาที่ "Your apps" → กดไอคอน **</>** (Web app) → ตั้งชื่อ nickname → Register app
3. จะได้โค้ดหน้าตาแบบนี้:
   ```js
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "math-bingo-xxxxx.firebaseapp.com",
     databaseURL: "https://math-bingo-xxxxx-default-rtdb.asia-southeast1.firebasedatabase.app",
     projectId: "math-bingo-xxxxx",
     storageBucket: "math-bingo-xxxxx.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef"
   };
   ```
4. คัดลอกโค้ดนี้ไปแทนที่ `firebaseConfig` **ทั้งสองไฟล์**:
   - `index.html` (มีคอมเมนต์ `▼▼▼ ใส่ Firebase Config ▼▼▼` บอกตำแหน่งไว้แล้ว)
   - `admin.html` (จุดเดียวกัน)

   ⚠️ ต้องใส่ config **ชุดเดียวกัน** ในทั้งสองไฟล์ ไม่งั้นครูกับนักเรียนจะคุยกันคนละฐานข้อมูล

## ขั้นตอนที่ 3: (แนะนำ) ตั้งกฎความปลอดภัยเบื้องต้น

โหมด "test mode" จะเปิดให้ใครก็เขียน/อ่านข้อมูลได้ และจะ**หมดอายุใน 30 วัน**
ถ้าจะใช้ต่อเนื่อง ให้ไปที่ Realtime Database > **Rules** แล้วตั้งเป็นแบบนี้ (เปิดอ่าน-เขียนตลอดไป เหมาะกับเกมในห้องเรียนที่ไม่มีข้อมูลอ่อนไหว) — ต้องเพิ่ม node `players` เข้าไปด้วย เพราะเวอร์ชันนี้มีระบบห้องรอ (lobby):
```json
{
  "rules": {
    "roomStatus": {
      ".read": true,
      ".write": true
    },
    "players": {
      ".read": true,
      ".write": true
    }
  }
}
```

## ขั้นตอนที่ 4: อัปโหลดขึ้น GitHub

1. สร้าง repository ใหม่บน GitHub (เช่น `math-bingo`)
2. อัปโหลดไฟล์ `index.html` และ `admin.html` ขึ้นไป (ไม่ต้องเอา `Code.gs` ไปด้วย)
3. ไปที่ **Settings > Pages** ของ repo
   - Source เลือก **Deploy from a branch**
   - Branch เลือก `main` (หรือ `master`) / folder `root` → Save
4. รอสักครู่ จะได้ลิงก์แบบ
   `https://ชื่อผู้ใช้.github.io/math-bingo/`

## ขั้นตอนที่ 5: ใช้งาน

- **ครู** เปิด: `https://ชื่อผู้ใช้.github.io/math-bingo/admin.html`
- **นักเรียน** เปิด: `https://ชื่อผู้ใช้.github.io/math-bingo/index.html`

(ต่างจากเดิมที่ใช้ `?page=admin` ต่อท้ายลิงก์เดียวกัน — เวอร์ชันนี้แยกเป็นคนละไฟล์/คนละลิงก์ไปเลย เพราะ static hosting ไม่รองรับ logic แบบ `doGet` ของ Apps Script)

---

### สิ่งที่เปลี่ยนจากเวอร์ชันเดิม
- ลบการพึ่งพา Google Apps Script (`Code.gs`, `google.script.run`, `PropertiesService`) ออกทั้งหมด
- `index.html` เปลี่ยนจาก polling ทุก 2 วินาที → ฟังข้อมูลแบบ real-time ผ่าน Firebase (`roomRef.on('value', ...)`) ทำให้เร็วขึ้นและลดโหลด
- เพิ่มไฟดวงเล็กๆ บอกสถานะการเชื่อมต่อ (เชื่อมต่อสำเร็จ/กำลังเชื่อมต่อ) ทั้งสองหน้า
- ตรรกะเกม (สุ่มบอร์ด, เช็คบิงโก, เสียง) เหมือนเดิมทุกอย่าง

### ฟีเจอร์ใหม่: ห้องรอผู้เล่น (Lobby)
ลำดับขั้นตอนตอนนี้เปลี่ยนเป็น:
1. ครูกรอกจำนวนการ์ดสูงสุด แล้วกด **"เปิดห้องรอผู้เล่น"**
2. หน้านักเรียนจะขึ้นปุ่ม **"พร้อมเล่น!"** — พอกดปุ่ม ระบบจะสุ่มชื่อเล่นให้อัตโนมัติ (เช่น "เสือขยัน42") แล้วส่งชื่อไปที่ห้องแอดมิน
3. หน้าแอดมินจะเห็นรายชื่อผู้เล่นที่เข้าห้องแบบเรียลไทม์ พร้อมจำนวนคนที่เข้าห้องแล้ว
4. เมื่อครูพร้อม กด **"เริ่มเกม"** — นักเรียนทุกคนจะไปหน้าเลือกจำนวนการ์ดพร้อมกัน
5. จากนั้นครูกด "สุ่มโจทย์ข้อต่อไป" เหมือนเดิม

ข้อมูลผู้เล่นเก็บที่ node `players` ใน Firebase แยกจาก `roomStatus` — ถ้านักเรียนปิดหน้าเว็บหรือหลุดเน็ต ชื่อจะหายไปจากรายชื่อโดยอัตโนมัติ (ใช้ `onDisconnect().remove()`) และเมื่อครูกด "รีเซ็ต" หรือ "เปิดห้องรอผู้เล่น" ใหม่ รายชื่อเก่าจะถูกล้างทิ้งทุกครั้ง
