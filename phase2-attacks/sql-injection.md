# SQL Injection — DVWA + Splunk

## เป้าหมาย
- ทดสอบ SQL Injection บน DVWA
- ดึงข้อมูลจาก Database
- วิเคราะห์ Log ที่เกิดขึ้นใน Splunk

---

## Environment
| Component | Details |
|-----------|---------|
| Attacker | Kali Linux |
| Target | DVWA SQL Injection Module |
| DVWA Security Level | Low |

---

## ขั้นตอน

### 1. เข้า DVWA → SQL Injection
> [บันทึกขั้นตอนตรงนี้]

### 2. ทดสอบ Basic SQLi
```sql
' OR '1'='1
```

### 3. ดู Log ใน Splunk
```
index=* sourcetype=* "SELECT" OR "UNION"
```

---

## ผลลัพธ์
> [ใส่ Screenshot ตรงนี้]

---

## สิ่งที่เรียนรู้
> [บันทึกสิ่งที่เรียนรู้ตรงนี้]
