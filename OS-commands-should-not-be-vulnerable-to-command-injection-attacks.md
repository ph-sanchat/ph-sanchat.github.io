# OS commands should not be vulnerable to command injection attacks

## คำสั่ง OS ไม่ควรเสี่ยงต่อการโจมตีด้วยการเพิ่มคำสั่ง

แอพพลิเคชั่นที่รันคำสั่งระบบปฏิบัติการหรือรันคำสั่งที่โต้ตอบกับระบบพื้นฐานควรปรับค่าที่จัดเตรียมจากภายนอกที่ใช้ในคำสั่งเหล่านั้นให้เป็นกลาง การไม่ทำเช่นนั้นอาจทำให้ผู้โจมตีสามารถใส่ข้อมูลที่เรียกใช้คำสั่งที่ไม่ได้ตั้งใจหรือเปิดเผยข้อมูลที่ละเอียดอ่อนได้

#### ปัญหาสามารถบรรเทาได้ด้วยวิธีใด ๆ ต่อไปนี้ :
* การใช้โมดูลกระบวนการย่อยโดยไม่มี shell = true ในกรณีนี้คาดว่ากระบวนการย่อยอาร์เรย์ที่คำสั่งและข้อโต้แย้งจะถูกแยกออกอย่างชัดเจน
* การหลีกเลี่ยงอาร์กิวเมนต์เชลล์ด้วย shlex.quote

### ตัวอย่างรหัสที่ไม่เป็นไปตามข้อกำหนด
* os
<pre>from flask import request
import os

@app.route('/ping')
def ping():
    address = request.args.get("address")
    cmd = "ping -c 1 %s" % address
    os.popen(cmd) # Noncompliant
</pre>

* subprocess
<pre>from flask import request
import subprocess

@app.route('/ping')
def ping():
    address = request.args.get("address")
    cmd = "ping -c 1 %s" % address
    subprocess.Popen(cmd, shell=True) # Noncompliant; using shell=true is unsafe
</pre>

### โซลูชันที่รองรับ
* os
<pre>from flask import request
import os

@app.route('/ping')
def ping():
    address = shlex.quote(request.args.get("address")) # address argument is shell-escaped
    cmd = "ping -c 1 %s" % address
    os.popen(cmd ) # Compliant
</pre>

* subprocess
<pre>from flask import request
import subprocess

@app.route('/ping')
def ping():
    address = request.args.get("address")
    args = ["ping", "-c1", address]
    subprocess.Popen(args) # Compliant
</pre>

### Reference : https://rules.sonarsource.com/python/type/Vulnerability/RSPEC-2076

### Author
* Sanchat Phaisit
