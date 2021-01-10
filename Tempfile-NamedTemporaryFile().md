# Tempfile.NamedTemporaryFile()

การสร้างไฟล์ชั่วคราวโดยใช้วิธีการที่ไม่ปลอดภัยจะทำให้แอปพลิเคชันตกอยู่ในสภาพการแข่งขันกับชื่อไฟล์ : ผู้ใช้ที่เป็นอันตรายสามารถพยายามสร้างไฟล์ที่มีชื่อที่คาดเดาได้ก่อนที่แอปพลิเคชันจะทำ การโจมตีที่ประสบความสำเร็จอาจส่งผลให้ไฟล์อื่นถูกเข้าถึงแก้ไขเสียหายหรือถูกลบ ความเสี่ยงนี้จะยิ่งสูงขึ้นหากแอปพลิเคชันทำงานด้วยสิทธิ์ที่สูงขึ้น

### ตัวอย่างรหัสที่ไม่เป็นไปตามข้อกำหนด
<pre>import tempfile

filename = tempfile.mktemp() # Noncompliant
tmp_file = open(filename, "w+")
</pre>

### Solution ที่รองรับ
<pre>import tempfile

tmp_file1 = tempfile.NamedTemporaryFile(delete=False) # Compliant; Easy replacement to tempfile.mktemp()
tmp_file2 = tempfile.NamedTemporaryFile() # Compliant; Created file will be automatically deleted
</pre>

#### Reference : https://rules.sonarsource.com/python/type/Vulnerability/RSPEC-5445

##### Author 
* Sanchat Phaisit
