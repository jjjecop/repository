# Brute Force Attack — DVWA + Splunk

## เป้าหมาย
- ทดสอบการโจมตี Brute Force บน Login Page ของ DVWA
- วิเคราะห์ Log ที่เกิดขึ้นใน Splunk
- เข้าใจ Pattern ของการโจมตีประเภทนี้

---

## Environment
| Component | Details |
|-----------|---------|
| Attacker | Kali Linux (172.16.143.131) |
| Target | DVWA Login Page |
| DVWA Security Level | Low |
| SIEM | Splunk (macOS) |

---

## ขั้นตอน

### 1. ตั้งค่า DVWA
- เข้า DVWA → DVWA Security → ตั้งเป็น **Low**

### 2. Fird URL DVWA for Login
- ก่อน Login URL [http://172.16.143.131/DVWA/vulnerabilities/brute/]

- หลังลอง Login URL มั่วๆ [http://172.16.143.131/DVWA/vulnerabilities/brute/?username=admin&password=password11&Login=Login#]  
> [ทำไปเพราะต้องการดูว่า URL หน้าตาเป็นยังไง เพื่อบอก Hydra:]
     - ส่ง request ไปที่ไหน
     - ใช้ parameter อะไรส่ง
     - เมื่อ login ผิดเว็บจะตอบอะไร

### 3. ทดสอบ Brute Force
> 

### 3. ดู Log ใน Splunk


```
index=* sourcetype=* "login"
```

---

## ผลลัพธ์
> [ใส่ Screenshot และผลลัพธ์ที่เห็นตรงนี้]

---

## สิ่งที่เรียนรู้
> [บันทึกสิ่งที่เรียนรู้ตรงนี้]
