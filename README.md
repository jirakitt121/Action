# webdev-actions-lab — LAB: GitHub Actions + GitHub Pages

เว็บไซต์ Static (HTML/CSS/JavaScript) ที่ Deploy อัตโนมัติไปยัง GitHub Pages ด้วย GitHub Actions

- Repository: https://github.com/jirakitt121/Action
- GitHub Pages: https://jirakitt121.github.io/Action/

## โครงสร้างไฟล์

```
├── index.html
├── style.css
├── script.js
└── .github/
    └── workflows/
        └── deploy.yml
```

## แบบฝึกหัด

- **Exercise 1:** ใส่ชื่อ–รหัสนิสิต และข้อความต้อนรับของตนเองใน `index.html`
- **Exercise 2:** หน้าเว็บมี 3 ส่วน คือ About Me, Skills และ Contact
- **Exercise 3:** ปุ่ม `Say Hello` เปลี่ยนข้อความ และปุ่ม `Dark Mode` สลับธีมสว่าง/มืด
- **Challenge:** ใน `deploy.yml` มีขั้นตอน `Check required files` ใช้ `test -f` ตรวจว่ามี `index.html`, `style.css`, `script.js` ก่อน Deploy ถ้าไม่มีไฟล์ใด Job จะ Fail และไม่ Deploy

### Exercise 4: Workflow เริ่มทำงานเมื่อใด และ Job deploy ทำอะไร

Workflow จะเริ่มทำงานใน 2 กรณี คือ (1) เมื่อมีการ `push` ขึ้น branch `main` และ (2) เมื่อกดรันเองจากแท็บ Actions (`workflow_dispatch`)

Job `deploy` จะรันบนเครื่อง `ubuntu-latest` แล้วทำตามลำดับ:
1. **Checkout** — ดึงโค้ดล่าสุดจาก Repository มาไว้บนเครื่อง Runner
2. **Check required files** — ตรวจว่ามีไฟล์หลักของเว็บครบ
3. **Setup Pages** — ตั้งค่าให้พร้อม Deploy ไปยัง GitHub Pages
4. **Upload static website** — รวมไฟล์ในโฟลเดอร์ (`.`) เป็น Artifact
5. **Deploy to GitHub Pages** — นำ Artifact ขึ้นเป็นเว็บไซต์ และได้ URL ของหน้าเว็บ

`permissions` ให้สิทธิ์เท่าที่จำเป็น (อ่านโค้ด, เขียน Pages, ขอ id-token) และ `concurrency` ทำให้ถ้ามีการ push ซ้อนกัน จะยกเลิกรอบเก่าและ Deploy เฉพาะรอบล่าสุด

## คำถามท้ายแลป

**1) GitHub Actions แตกต่างจาก GitHub Pages อย่างไร?**
GitHub Actions คือระบบรันงานอัตโนมัติ (CI/CD) เมื่อเกิดเหตุการณ์ เช่น push ส่วน GitHub Pages คือบริการโฮสต์เว็บไซต์ Static ให้เข้าถึงผ่าน URL สาธารณะ ในแลปนี้ Actions เป็น "คนส่งของ" ที่นำไฟล์ไปขึ้น Pages ซึ่งเป็น "ที่ตั้งเว็บ"

**2) เหตุใดจึงเลือก trigger ด้วย push ที่ branch main?**
เพราะ `main` เป็น branch หลักที่เก็บโค้ดเวอร์ชันพร้อมใช้งาน การ Deploy เฉพาะเมื่อ push เข้า `main` ทำให้เว็บจริงอัปเดตตามโค้ดที่ผ่านการตรวจแล้ว ส่วนงานที่ยังทดลองใน branch อื่นจะไม่ไปกระทบเว็บจริง

**3) ถ้าแก้ไฟล์ในเครื่องแต่ไม่ได้ push เว็บไซต์จะเปลี่ยนหรือไม่ เพราะเหตุใด?**
ไม่เปลี่ยน เพราะ GitHub ยังไม่ได้รับโค้ดใหม่ จึงไม่มี event `push` ไปกระตุ้น Workflow และ GitHub Pages ยังแสดงไฟล์ชุดที่ Deploy ไว้ล่าสุด การ commit อย่างเดียวก็ยังอยู่แค่ในเครื่อง ต้อง push จึงจะเริ่ม Deploy

**4) เหตุใดโปรเจกต์ PHP + MySQL จึงไม่สามารถนำไปรันบน GitHub Pages ได้โดยตรง?**
GitHub Pages ส่งไฟล์ Static ให้เบราว์เซอร์เท่านั้น ไม่มีตัวแปลภาษา PHP ฝั่ง Server และไม่มีฐานข้อมูล MySQL ไฟล์ `.php` จึงไม่ถูกประมวลผล ต้องใช้ Hosting ที่รองรับ PHP/MySQL หรือแยกเฉพาะส่วนหน้าเว็บมาไว้บน Pages

**5) อธิบายขั้นตอนการค้นหาสาเหตุ เมื่อ Workflow มีสถานะ Failed**
1. เปิดแท็บ Actions → เลือก Run ที่ Failed → เลือก Job `deploy`
2. ดูว่า Step ใดมีเครื่องหมาย ❌ แล้วกางดู Log เพื่ออ่านข้อความ Error
3. จัดกลุ่มสาเหตุ เช่น YAML ผิด (การเยื้อง), หาไฟล์ไม่เจอ (`test -f` ไม่ผ่าน), สิทธิ์ไม่พอ (Settings → Pages ยังไม่ได้เลือก GitHub Actions หรือ `permissions` ผิด), หรือ branch ไม่ใช่ `main`
4. แก้ไขในเครื่อง → commit → push ใหม่ (หรือกด Re-run jobs) แล้วตรวจว่าเป็นสีเขียว และเปิดเว็บยืนยันผล
