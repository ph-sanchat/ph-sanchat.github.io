# Logging should not be vulnerable to injection attacks

## เข้าสู่ระบบไม่ควรจะเสี่ยงต่อการโจมตี

ผู้ใช้บริการที่มีให้ข้อมูลเช่นจำกัด URL ข้อมูลหรือคุกกี้ควรได้รับการพิจารณาที่ไม่น่าเชื่อถือและการปนเปื้อน แอปพลิเคชั่นบันทึกข้อมูลที่ปนเปื้อนอาจทำให้ผู้โจมตีจะทำลายรูปแบบไฟล์บันทึกได้ ซึ่งอาจจะใช้ในการตรวจสอบป้องกันและ SIEM (ข้อมูลการรักษาความปลอดภัยและการจัดการเหตุการณ์) ระบบตรวจจับเหตุการณ์ที่เป็นอันตรายอื่น ๆ

ปัญหานี้อาจจะลดลงโดยการล้างข้อมูลของผู้ใช้บริการที่มีให้ก่อนที่จะเข้าสู่ระบบ

### ตัวอย่างรหัสที่ไม่เป็นไปตามข้อกำหนด
<pre>from flask import request, current_app
import logging

@app.route('/log')
def log():
    input = request.args.get('input')
    current_app.logger.error("%s", input) # Noncompliant
</pre>

### โซลูชันที่รองรับ
<pre>from flask import request, current_app
import logging

@app.route('/log')
def log():
    input = request.args.get('input')
    if input.isalnum():
        current_app.logger.error("%s", input) # Compliant
</pre>

### Reference : https://rules.sonarsource.com/python/type/Vulnerability/RSPEC-5145

### Author
* Sanchat Phaisit
