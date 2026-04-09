## การป้องกัน (Recommendation)

### 1. ใช้ Prepared Statements / Parameterized Queries
แทนที่จะนำ input ของ user ไปต่อใน SQL query โดยตรง ให้ใช้ parameterized query เช่น
```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$id]);
```

### 2. Validate และ Sanitize Input
- ตรวจสอบว่า input เป็น type ที่ถูกต้อง เช่น ถ้าต้องการตัวเลขก็ต้อง int เท่านั้น
- ปฏิเสธ input ที่มี special character เช่น `'`, `"`, `--`, `;`

### 3. ใช้ Least Privilege สำหรับ Database User
- Account ที่ web app ใช้เชื่อมต่อ DB ควรมีสิทธิ์แค่ที่จำเป็น
- ไม่ควรใช้ `root` หรือ account ที่มีสิทธิ์ DROP / DELETE

### 4. ปิด Error Message ที่ละเอียดเกินไป
- อย่าแสดง SQL error ออกหน้าเว็บ เพราะ attacker จะรู้โครงสร้าง DB
- ใช้ generic error message แทน เช่น "Something went wrong"

### 5. ใช้ Web Application Firewall (WAF)
- WAF ช่วย detect และ block SQL Injection pattern อัตโนมัติ
- เช่น ModSecurity, AWS WAF, Cloudflare WAF

### 6. Monitor ด้วย Splunk
- ตั้ง Alert ใน Splunk เมื่อพบ keyword น่าสงสัยใน log เช่น `SELECT`, `UNION`, `DROP`
- ตัวอย่าง query:
```
index=* sourcetype=* ("SELECT" OR "UNION" OR "DROP" OR "INSERT")
| stats count by src_ip, uri
| where count > 10
```