# XSS (Cross-Site Scripting) — DVWA + Splunk

## เป้าหมาย
- ทดสอบ XSS บน DVWA
- เข้าใจวิธีการฝัง Script ในเว็บ
- วิเคราะห์ Log ใน Splunk

---

## ขั้นตอน

### 1. เข้า DVWA → XSS (Reflected)
> [บันทึกขั้นตอนตรงนี้]

### 2. ทดสอบ Basic XSS
```html
<script>alert('XSS')</script>
```

### 3. ดู Log ใน Splunk
```
index=* sourcetype=* "script"
```

---

## ผลลัพธ์
> [ใส่ Screenshot ตรงนี้]

---

## สิ่งที่เรียนรู้
> [บันทึกสิ่งที่เรียนรู้ตรงนี้]
