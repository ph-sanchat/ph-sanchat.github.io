# Casting to safe types

### การดำเนินการแก้ไขรหัส ไม่ควรเสี่ยงต่อ injection attacks

  แอปพลิเคชั่นที่ run โค้ดแบบไดนามิกควรทำให้ค่าที่ได้รับจากภายนอกที่ใช้สร้างโค้ดเป็นกลาง หากไม่ดำเนินการดังกล่าวอาจทำให้ผู้โจมตีสามารถเรียกใช้รหัสได้โดยง่าย สิ่งนี้สามารถเปิดใช้งานการโจมตีที่รุนแรงได้หลายรูปแบบเช่นการเข้าถึง / แก้ไขข้อมูลที่ละเอียดอ่อนหรือเข้าถึงระบบทั้งหมด

### ตัวอย่างรหัสที่ไม่เป็นไปตามข้อกำหนด
<pre>from flask import request

@app.route('/')
def index():
    module = request.args.get("module")
    exec("import urllib%s as urllib" % module) # Noncompliant
</pre>

### โซลูชันที่รองรับ
<pre>from flask import request

@app.route('/')
def index():
    module = request.args.get("module")
    exec("import urllib%d as urllib" % int(module)) # Compliant; module is safely cast to an integer
</pre>

#### Reference : https://rules.sonarsource.com/python/type/Vulnerability/RSPEC-5334

#### Author
* Sanchat Phaisit
