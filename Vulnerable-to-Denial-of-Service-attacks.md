# Regular expressions should not be vulnerable to Denial of Service attacks

** ส่วนใหญ่ของการแสดงออกปกติเครื่องมือใช้ย้อนรอยเส้นทางที่จะพยายามดำเนินการทั้งหมดเป็นไปได้ของการแสดงออกปกติเมื่อมีการประเมินการป้อนข้อมูลในบางกรณีก็อาจทำให้เกิดปัญหาประสิทธิภาพการทำงานที่เรียกว่าสถานการณ์ภัยพิบัติย้อนรอย<br>
** ในกรณีที่เลวร้ายที่สุดความซับซ้อนของการแสดงออกปกติคือชี้แจงในขนาดของการป้อนข้อมูลที่นี้หมายถึงว่ามีขนาดเล็กอย่างรอบคอบสร้างขึ้นมาป้อนข้อมูล (เช่น 20 ตัวอักษร) สามารถเรียกย้อนรอยหายนะและทำให้เกิดการปฏิเสธการให้บริการของแอพลิเคชัน<br>
** ความซับซ้อนของการแสดงออกขั้นสูงสามารถนำไปสู่ผลกระทบเดียวกันได้เช่นกันในกรณีนี้อินพุตที่สร้างขึ้นอย่างพิถีพิถันขนาดใหญ่ (หลายพันตัวอักษร)

** ไม่แนะนำให้สร้างรูปแบบนิพจน์ทั่วไปจากอินพุตที่ผู้ใช้ควบคุมหากไม่มีทางเลือกอื่นให้ล้างข้อมูลอินพุตเพื่อลบ / ทำลาย regex

### ตัวอย่างรหัสที่ไม่เป็นไปตามข้อกำหนด
<pre>from flask import request
import re

@app.route('/upload')
def upload():
    username = request.args.get('username')
    filename = request.files.get('attachment').filename

    re.search(username, filename) # Noncompliant

</pre>

### โซลูชันที่รองรับ
<pre>from flask import request
import re

@app.route('/upload')
def upload():
    username = re.escape(request.args.get('username'))
    filename = request.files.get('attachment').filename

    re.search(username, filename) # Compliant

</pre>

### Reference : https://rules.sonarsource.com/python/type/Vulnerability/RSPEC-2631

### Author
* Sanchat Phaisit
