## การป้องกัน (Recommendation)

### 1. Account Lockout
ล็อค account หลังพิมพ์ password ผิดเกิน 5 ครั้ง
ทำให้ Hydra ยิงต่อไม่ได้ ต้องรอ unlock

### 2. Rate Limiting
จำกัด login ได้ไม่เกิน 5 ครั้ง/นาที
ทำให้ Hydra ช้าลงมาก ไม่คุ้มที่จะโจมตี

### 3. CAPTCHA
เพิ่มการทดสอบว่าเป็นคนไม่ใช่ Bot
Hydra เป็นโปรแกรม ไม่สามารถผ่าน CAPTCHA ได้

### 4. Monitor ด้วย Splunk
ตั้ง Alert เมื่อมี IP ส่ง request มากกว่า 5 ครั้ง
ทำให้รู้ทันทีเมื่อมีการโจมตีเกิดขึ้น