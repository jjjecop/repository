# Phase 1 — Environment Setup

## เป้าหมาย
- ติดตั้งและเชื่อมต่อ DVWA บน Kali Linux กับ Splunk บน macOS
- เรียนรู้การอ่าน Log เบื้องต้นใน Splunk
- ทำความเข้าใจ Flow: DVWA → Apache Log → Splunk Forwarder → Splunk

---

## Environment
| Component | Details |
|-----------|---------|
| Host OS | macOS (MacBook Pro M2) |
| VM OS | Kali Linux ARM64 (VMware Fusion) |
| Kali IP | 172.16.x.x (VMware NAT) |
| Mac IP  | 192.168.x.x (VMware Gateway) |
| Target App | DVWA (/var/www/html/DVWA) |
| SIEM | Splunk Enterprise (localhost:8000) |

---

## ขั้นตอนที่ทำ

### 1. เริ่ม Services บน Kali
```bash
sudo service apache2 start
sudo service mysql start
```

### 2. เข้าใช้งาน DVWA
- จาก Kali: `http://localhost/DVWA`
- จาก Mac: `http://172.16.143.131/DVWA`
- Login: admin / password

> หมายเหตุ: Linux เป็น Case-Sensitive ต้องพิมพ์ DVWA ตัวใหญ่

### 3. ติดตั้ง Splunk Universal Forwarder บน Kali
```bash
# ดาวน์โหลด ARM .deb (สำหรับ M2)
wget -O splunkforwarder.deb "[wget link จาก splunk.com]"

# ติดตั้ง
sudo dpkg -i splunkforwarder-[version]-linux-arm64.deb

# Start ครั้งแรก
sudo /opt/splunkforwarder/bin/splunk start --accept-license
```

### 4. Config Forwarder ส่ง Log ไปที่ Mac
```bash
# เชื่อมต่อกับ Splunk บน Mac (Port 9997)
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.154.1:9997 -auth admin:[password]

# Monitor Apache Log ของ DVWA
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/apache2/ -auth admin:[password]
```

### 5. เปิด Port 9997 บน Splunk (Mac)
```bash
splunk enable listen 9997 -auth admin:[password]
```

---

## ผลลัพธ์

### Splunk Search
```
index=* sourcetype=*
```

**ผลที่เห็น:** 54 Events จาก Kali

| Field | Value |
|-------|-------|
| host | kali |
| source | /var/log/apache2/access.log |
| sourcetype | access-too-small |

### ตัวอย่าง Log ที่เห็น
```
172.16.143.1 - - [14/Mar/2026:13:05:49 +0700] "GET /DVWA/index.php HTTP/1.1" 200 2799
```

**อ่าน Log ได้ว่า:**
- `172.16.143.1` = IP ของ Mac ที่เข้ามา
- `GET /DVWA/index.php` = เข้าหน้า index
- `200` = สำเร็จ
- `2799` = ขนาด Response (bytes)

---

## สิ่งที่เรียนรู้
- Apache Log บันทึก IP, Method, URL, Status Code, Response Size ทุก Request
- Splunk Universal Forwarder ทำหน้าที่ส่ง Log แบบ Real-time ข้าม OS
- Port 9997 คือ Port มาตรฐานของ Splunk สำหรับรับ Log จาก Forwarder
- Linux Case-Sensitive ทำให้ `localhost/dvwa` ไม่ทำงาน ต้องใช้ `localhost/DVWA`

---

### Splunk Search Query
```
index=* sourcetype=*
```
> ค้นหา Log ทั้งหมดที่ Splunk รับมาจาก Kali
> - `index=*` = ทุก index
> - `sourcetype=*` = ทุกประเภท Log

## Screenshot
![Splunk 72 Events](screenshots/splunk-events.png)

---

## Next Step → Phase 2
เริ่มโจมตี DVWA ด้วย Brute Force แล้วดูว่า Splunk จับ Log ได้แบบไหน
