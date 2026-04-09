# Phase 3 — Splunk Analysis

## เป้าหมาย
- สร้าง Dashboard แสดง Attack Pattern
- ตั้ง Alert เมื่อมีการโจมตีเกิดขึ้น
- เขียน Correlation Rule ตรวจจับ Brute Force

---

## Dashboard

### ขั้นตอนสร้าง Dashboard

1. เปิด Splunk → เมนูบน → **Search** → รัน query ด้านล่าง
2. คลิก **Save As** → **Dashboard Panel**
3. ตั้งชื่อ Dashboard: `DVWA Attack Monitor`
4. เลือก **Classic Dashboards**
5. ตั้งชื่อ Panel: `Brute Force Attacks by IP`
6. คลิก **Save to Dashboard**

### SPL Query — Brute Force Attacks by IP

```spl
index=main sourcetype="access-too_small"
| rex field=_raw "^(?<src_ip>\S+)"
| search _raw="*brute*"
| stats count by src_ip
| sort -count
```

### ผลลัพธ์

| src_ip | count |
|--------|-------|
| 172.16.143.1 | 40 |
| 172.16.143.131 | 24 |

- IP `172.16.143.1` (Kali) ส่ง request มายัง `/brute/` มากที่สุด 40 ครั้ง
- IP `172.16.143.131` (DVWA เอง) ปรากฏ 24 ครั้ง เป็น internal request
- Pattern ชัดเจนว่ามีการทำ Brute Force Attack จาก IP เดียว


![Dashboards](/phase3-splunk/screenshots/01.png)

---

## Alert

### ขั้นตอนตั้ง Alert

1. รัน query ด้านล่างใน Splunk Search
2. คลิก **Save As** → **Alert**
3. กรอกข้อมูลตามตารางด้านล่าง
4. คลิก **+ Add Actions** → **Add to Triggered Alerts**
5. คลิก **Save**

### SPL Query — Alert Trigger

```spl
index=main sourcetype="access-too_small"
| rex field=_raw "^(?<src_ip>\S+)"
| search _raw="*brute*"
| stats count by src_ip
| where count > 10
```

### การตั้งค่า Alert

| Setting | ค่า |
|---------|-----|
| Title | Brute Force Detected |
| Alert Type | Scheduled |
| Schedule | Run every hour |
| Trigger Condition | Number of Results > 0 |
| Trigger | Once |
| Action | Add to Triggered Alerts |
| Severity | Medium |

![Alert](/phase3-splunk/screenshots/02.png)

### วิธีตรวจสอบว่า Alert ทำงาน

ไปที่ **Activity** → **Triggered Alerts** → จะเห็น `Brute Force Detected` ปรากฏขึ้นเมื่อมีการโจมตีเกิดขึ้น

---

## Correlation Rule

```spl
index=main sourcetype="access-too_small"
| rex field=_raw "^(?<src_ip>\S+)"
| rex field=_raw "HTTP/\d\.\d\" (?<status>\d{3})"
| search _raw="*brute*"
| stats count as total_requests, dc(status) as unique_status by src_ip
| where total_requests > 5
| eval risk=case(
    total_requests > 30, "HIGH",
    total_requests > 10, "MEDIUM",
    true(), "LOW"
)
| table src_ip, total_requests, unique_status, risk
```

### ผลลัพธ์

| src_ip | total_requests | unique_status | risk |
|--------|---------------|---------------|------|
| 172.16.143.1 | 40 | 3 | **HIGH** |
| 172.16.143.131 | 24 | 2 | **MEDIUM** |

- IP `172.16.143.1` ถูกจัดระดับ **HIGH** เนื่องจากส่ง request มากกว่า 30 ครั้ง
- มี unique HTTP status codes 3 แบบ บ่งชี้ว่ามีทั้ง request ที่สำเร็จและล้มเหลว