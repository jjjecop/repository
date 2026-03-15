# Phase 3 — Splunk Analysis

## เป้าหมาย
- สร้าง Dashboard แสดง Attack Pattern
- ตั้ง Alert เมื่อมีการโจมตีเกิดขึ้น
- เขียน Correlation Rule ตรวจจับ Brute Force

---

## Dashboard
> [บันทึกขั้นตอนสร้าง Dashboard ตรงนี้]

---

## Alert
> [บันทึกการตั้ง Alert ตรงนี้]

---

## Correlation Rule
```
index=* sourcetype=* status=401
| stats count by src_ip
| where count > 5
```
> [บันทึกผลลัพธ์ตรงนี้]
