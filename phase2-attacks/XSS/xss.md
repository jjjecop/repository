# XSS (Cross-Site Scripting) — DVWA + Splunk

## เป้าหมาย
- ทดสอบ XSS บน DVWA
- เข้าใจวิธีการฝัง Script ในเว็บ
- วิเคราะห์ Log ใน Splunk

---

## Environment
| Component | Details |
|-----------|---------|
| Attacker | Kali Linux (172.16.143.1) |
| Target | DVWA — 172.16.143.131 |
| URL | http://172.16.143.131/DVWA/vulnerabilities/xss_r/ |
| DVWA Security Level | Low |
| Web Server | Apache2 |
| Log Source | /var/log/apache2/access.log |

---

## ทฤษฎี XSS คืออะไร
XSS (Cross-Site Scripting) คือการที่ attacker แทรก JavaScript เข้าไปในหน้าเว็บ เพื่อให้ browser ของ victim รัน script นั้น โดยที่ victim ไม่รู้ตัว

**ประเภทของ XSS:**
| ประเภท | ความหมาย |
|--------|----------|
| Reflected XSS | Script ถูกส่งผ่าน URL แล้วสะท้อนกลับมาในหน้าเว็บทันที |
| Stored XSS | Script ถูกบันทึกลง Database แล้วแสดงทุกครั้งที่เปิดหน้านั้น |
| DOM-based XSS | Script ถูกรันผ่าน DOM โดยไม่ผ่าน Server |

---

## รูปแบบการโจมตีและประโยชน์ที่ได้
การโจมตีครั้งนี้เป็นแบบ **Reflected XSS** โดย payload ถูกส่งผ่าน URL parameter `?name=` แล้วสะท้อนกลับมาในหน้าเว็บโดยไม่มีการ sanitize

**ประโยชน์ที่ attacker ได้รับ:**
| ประโยชน์ | รายละเอียด |
|---------|-----------|
| Proof of Concept | แสดงได้ว่าระบบมีช่องโหว่ XSS จริง |
| Cookie Stealing | ต่อยอดเป็น `document.cookie` เพื่อขโมย session |
| Phishing | แสดง popup หลอกให้ user กรอกข้อมูล |
| Redirect | redirect ไปยังเว็บอันตราย |

---

## ขั้นตอน

### 1. เข้า DVWA → XSS (Reflected)
เปิดหน้า XSS (Reflected) บน DVWA พบช่อง input **"What's your name?"**

![หน้า DVWA XSS Reflected](/phase2-attacks/XSS/screenshot/01_xss_reflected_page.png)

---

### 2. ทดสอบ Basic XSS

**Payload ที่ใช้:**
```html
<script>alert('XSS by Thawatchai')</script>
```

**วิธีพิมพ์:** กรอกใส่ช่อง "What's your name?" แล้วกด Submit

**เหตุผลที่ได้ผล:**
ระบบนำ input ไปแสดงในหน้าเว็บโดยตรงโดยไม่มีการ sanitize เช่น:
```html
Hello <script>alert('XSS by Thawatchai')</script>
```
Browser จึงตีความ `<script>` เป็น JavaScript แล้วรันทันที

**ผลลัพธ์:** ขึ้น popup alert "XSS by Thawatchai" จาก 172.16.143.131

![Popup Alert XSS by Thawatchai](screenshots/02_xss_alert_popup.png)

---

### 3. ดู Log ใน Splunk

**Query ที่ใช้:**
index=* sourcetype=access-too_small
**Log ที่พบ — วิเคราะห์:**
| ฟิลด์ | ค่าที่พบ |
|------|---------|
| Timestamp | 09/Apr/2026 20:04:49–20:04:56 |
| Source IP | 172.16.143.1 (Kali Linux) |
| Method | GET |
| URI | /DVWA/vulnerabilities/xss_r/?name=%3Cscript%3Ealert%28%27XSS+by+Thawatchai%27%29%3C%2Fscript%3E |
| Payload (decoded) | `<script>alert('XSS by Thawatchai')</script>` |
| HTTP Status | 200 (สำเร็จ) |
| Host | kali |
| Log file | /var/log/apache2/access.log |

> **หมายเหตุ:** `%3C` = `<`, `%3E` = `>`, `%27` = `'` คือ URL encode ของ HTML tag

![Splunk Log XSS](screenshots/03_xss_splunk_log.png)

---

## ผลลัพธ์สรุป
| การทดสอบ | Payload | ผลลัพธ์ |
|----------|---------|---------|
| Reflected XSS | `<script>alert('XSS by Thawatchai')</script>` | ขึ้น popup alert สำเร็จ |
| ตรวจพบใน Splunk | `index=* sourcetype=access-too_small` | พบ log การโจมตีพร้อม encoded payload ใน URI |

---

## สิ่งที่เรียนรู้
> - XSS เกิดจากระบบนำ input ของ user ไปแสดงในหน้าเว็บโดยไม่มีการ sanitize หรือ encode
> - Payload `<script>alert('XSS by Thawatchai')</script>` ทำให้ browser รัน JavaScript ได้ทันที
> - Log ใน Apache2 บันทึก payload ไว้ในรูป URL encode เช่น `%3Cscript%3E`
> - Reflected XSS อันตรายมากเมื่อนำไปต่อยอดเป็นการขโมย Cookie หรือ Session

---

## Conclusion
ระบบ DVWA ระดับ Low ไม่มีการป้องกัน XSS ใดๆ ทำให้ attacker สามารถฝัง JavaScript เข้าไปใน URL แล้วหลอกให้ victim คลิก เพื่อขโมย session หรือ redirect ไปยังเว็บอันตรายได้ Splunk สามารถตรวจจับได้จาก URI ที่มี HTML tag ในรูป URL encode

---

## References
- [OWASP XSS](https://owasp.org/www-community/attacks/xss/)
- [DVWA GitHub](https://github.com/digininja/DVWA)
- [Splunk Search Reference](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference)