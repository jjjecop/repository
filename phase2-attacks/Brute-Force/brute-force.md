# Brute Force Attack — DVWA + Splunk

## เป้าหมาย
- ทดสอบการโจมตี Brute Force บน Login Page ของ DVWA
- วิเคราะห์ Log ที่เกิดขึ้นใน Splunk
- เข้าใจ Pattern ของการโจมตีประเภทนี้

---

## Environment
| Component | Details |
|-----------|---------|
| Attacker | Kali Linux (172.16.143.1) |
| Target | DVWA (172.16.143.131) |
| DVWA Security Level | Low |
| SIEM | Splunk (macOS) |

---

## ขั้นตอน

### 1. ตั้งค่า DVWA
- เข้า DVWA → DVWA Security → ตั้งเป็น **Low**

### 2. หา URL ของ DVWA Login
- ก่อน Login URL: `http://172.16.143.131/DVWA/vulnerabilities/brute/`
- หลังลอง Login URL มั่วๆ: `http://172.16.143.131/DVWA/vulnerabilities/brute/?username=admin&password=password11&Login=Login#`

> ทำไปเพราะต้องการดูว่า URL หน้าตาเป็นยังไง เพื่อบอก Hydra:
> - ส่ง request ไปที่ไหน
> - ใช้ parameter อะไรส่ง
> - เมื่อ login ผิด เว็บจะตอบอะไร

### 3. ทดสอบ Brute Force

#### Tool: Hydra v9.6

![Hydra Result](Brute-Force/screenshots/01_brute_force_hydra_result.png)

**Figure 1: Hydra Brute Force Attack Result**

Hydra สามารถ crack password ของ admin ได้สำเร็จภายใน 1 วินาที
โดยใช้ wordlist rockyou.txt ที่มี password 14,344,399 รายการ
DVWA Security Level = Low ไม่มี rate limiting หรือ account lockout
ทำให้โจมตีได้ง่าย

| รายละเอียด | ข้อมูล |
|------------|--------|
| Tool | Hydra v9.6 |
| Target | 172.16.143.131 (DVWA) |
| Username | admin |
| Password ที่เจอ | password, 123456, 12345, 123456789 |
| Duration | ~1 วินาที |

### 4. ดู Log ใน Splunk

#### Splunk Query: Brute Force Detection
```spl
index=* sourcetype="access-too_small" brute
| rex field=_raw "^(?<src_ip>\S+)"
| stats count by src_ip
| sort -count
```

**อธิบาย Query:**

| บรรทัด | ความหมาย |
|--------|----------|
| `index=* sourcetype="access-too_small" brute` | ค้นหา Log ทั้งหมดที่มีคำว่า "brute" |
| `rex field=_raw "^(?<src_ip>\S+)"` | ดึง IP address ออกจาก Log ดิบ |
| `stats count by src_ip` | นับจำนวน request แยกตาม IP |
| `sort -count` | เรียงจากมากไปน้อย |

![Splunk Brute Force](Brute-Force/screenshots/02_splunk_brute_force.png)

**Figure 2: Splunk Detection Result**

พบ IP 172.16.143.1 ส่ง request มายัง /brute/
จำนวน 27 ครั้ง ซึ่งเป็นพฤติกรรมผิดปกติ
บ่งชี้ว่ามีการทำ Brute Force Attack

### 5. Dashboard

![Dashboard](Brute-Force/screenshots/03_dashboard_brute_force.png)

**Figure 3: DVWA Attack Monitor Dashboard**

Dashboard แสดง Brute Force Attacks by IP
เห็น IP 172.16.143.1 ส่ง request มามากที่สุด

---

## ผลลัพธ์
- Hydra crack password สำเร็จภายใน 1 วินาที
- Password จริงของ admin คือ **password**
- Splunk ตรวจพบ 51 events จาก IP 172.16.143.1
- Dashboard แสดง pattern การโจมตีชัดเจน

---

## สิ่งที่เรียนรู้
- DVWA Security Level = Low ไม่มีการป้องกัน Brute Force เลย
- Hydra สามารถ crack password ได้ภายใน 1 วินาที
- Apache Log บันทึก User-Agent "Hydra" ทำให้ detect ได้ง่าย
- IP เดียวส่ง request ซ้ำๆ จำนวนมากเป็น pattern ของ Brute Force