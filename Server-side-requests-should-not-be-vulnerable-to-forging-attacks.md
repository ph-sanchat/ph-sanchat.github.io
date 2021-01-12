### คำขอฝั่งเซิร์ฟเวอร์ไม่ควรเสี่ยงต่อการปลอมแปลงการโจมตี

ผู้ใช้บริการที่มีให้ข้อมูลเช่นจำกัด URL ข้อมูลหรือคุกกี้ควรได้รับการพิจารณาที่ไม่น่าเชื่อถือและการปนเปื้อน เซิร์ฟเวอร์ระยะไกลที่ส่งคำขอไปยัง URL ตามข้อมูลที่ปนเปื้อนอาจทำให้ผู้โจมตีสามารถส่งคำขอไปยังเครือข่ายภายในหรือระบบไฟล์ภายในได้โดยพลการ

#### ปัญหาสามารถบรรเทาได้ด้วยวิธีใด ๆ ต่อไปนี้
* ตรวจสอบความถูกต้องของข้อมูลที่ผู้ใช้ให้มาตามรายการที่อนุญาตและปฏิเสธอินพุตที่ไม่ตรงกัน
* ออกแบบแอปพลิเคชันใหม่เพื่อไม่ให้ส่งคำขอตามข้อมูลที่ผู้ใช้ให้มา

### ตัวอย่างรหัสที่ไม่เป็นไปตามข้อกำหนด
<pre>from flask import request
import urllib

@app.route('/proxy')
def proxy():
    url = request.args["url"]
    return urllib.request.urlopen(url).read() # Noncompliant
</pre>

### โซลูชันที่รองรับ
<pre>from flask import request
import urllib

DOMAINS_WHITELIST = ['domain1.com', 'domain2.com']

@app.route('/proxy')
def proxy():
    url = request.args["url"]
    if urllib.parse.urlparse(url).hostname in DOMAINS_WHITELIST:
        return urllib.request.urlopen(url).read() # Compliant
</pre>

### Reference : https://rules.sonarsource.com/python/type/Vulnerability/RSPEC-5144

### Author
* Sanchat Phaisit
