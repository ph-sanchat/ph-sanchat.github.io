# LDAP Injection

![](https://dpsvdv74uwwos.cloudfront.net/statics/img/blogposts/ldap-injection.png)

การโจมตีด้วย `LDAP Injection` ใช้ช่องโหว่ในการตรวจสอบความถูกต้องของอินพุตเพื่อ Inject และดำเนินการสืบค้นไปยังเซิร์ฟเวอร์ Lightweight Directory Access Protocol บริการ LDAP มีความสำคัญต่อการดำเนินงานประจำวันของหลายองค์กรและการโจมตีด้วย LDAP Injection ที่ประสบความสำเร็จสามารถให้ข้อมูลที่มีค่าสำหรับการโจมตีฐานข้อมูลและแอปพลิเคชันภายในเพิ่มเติม บทความนี้กล่าวถึงวิธีการทำงานของ LDAP Injection และแสดงวิธีป้องกันเพื่อปรับปรุงความปลอดภัยของเว็บแอปพลิเคชัน

### ความสำคัญของเซิร์ฟเวอร์ LDAP

Lightweight Directory Access Protocol หรือ LDAP เป็นโปรโตคอลแอปพลิเคชันแบบเปิดสำหรับการเข้าถึงและบำรุงรักษาบริการไดเร็กทอรีในเครือข่าย IP (ดูข้อกำหนด RFC 4511) โดยทั่วไปองค์กรจะจัดเก็บข้อมูลเกี่ยวกับผู้ใช้และทรัพยากรในไดเร็กทอรีกลาง (เช่น Active Directory) และแอปพลิเคชันสามารถเข้าถึงและจัดการข้อมูลนี้โดยใช้คำสั่ง LDAP เซิร์ฟเวอร์ LDAP เป็นประตูสู่ข้อมูลที่ละเอียดอ่อนมากมายรวมถึงข้อมูลรับรองผู้ใช้ชื่อพนักงานและบทบาทอุปกรณ์ทรัพยากรเครือข่ายที่ใช้ร่วมกันเป็นต้น แม้ว่าจะมีการเผยแพร่สู่สาธารณะน้อยกว่าการโจมตีด้วยการฉีด SQL แต่การโจมตีด้วยการฉีด LDAP สามารถให้ข้อมูลที่มีค่าเกี่ยวกับโครงสร้างพื้นฐานภายในขององค์กรและอาจทำให้ผู้โจมตีสามารถเข้าถึงเซิร์ฟเวอร์ฐานข้อมูลและระบบภายในอื่น ๆ ได้

### LDAP Statement Syntax

ไคลเอ็นต์สามารถสอบถามเซิร์ฟเวอร์ LDAP ได้โดยการส่งคำร้องขอรายการไดเร็กทอรีที่ตรงกับตัวกรองเฉพาะ หากพบรายการที่ตรงกับตัวกรองการค้นหา LDAP เซิร์ฟเวอร์จะส่งคืนข้อมูลที่ร้องขอ ตัวกรองการค้นหาที่ใช้ในการสืบค้น LDAP เป็นไปตามไวยากรณ์ที่ระบุใน RFC 4515 (เดิมคือ RFC 2254) ตัวกรองสร้างจากแอตทริบิวต์ LDAP จำนวนเท่าใดก็ได้ที่ระบุเป็นคู่คีย์ - ค่าในวงเล็บ ตัวกรองสามารถรวมกันได้โดยใช้ตัวดำเนินการเชิงตรรกะและตัวดำเนินการเปรียบเทียบและสามารถรวมสัญลักษณ์แทนได้ นี่คือตัวอย่างบางส่วน:

- (cn = John *) จับคู่รายการที่ชื่อสามัญขึ้นต้นด้วย John (* ตรงกับอักขระใด ๆ )
- (! (cn = * Doe)) จับคู่รายการที่ชื่อสามัญไม่ได้ลงท้ายด้วย Doe (! is logical NOT)
- (& (cn = J *) (cn = * Doe)) จับคู่รายการที่ชื่อสามัญขึ้นต้นด้วย J และลงท้ายด้วย Doe (& เป็นตรรกะ AND)
- (& (| (cn = John *) (cn = Jane *)) (cn = * Doe)) จับคู่รายการที่ชื่อสามัญขึ้นต้นด้วย John หรือ Jane และลงท้ายด้วย Doe (| เป็นตรรกะหรือ)
ตัวกรองและตัวดำเนินการหลายตัวถูกรวมเข้าด้วยกันโดยใช้สัญกรณ์นำหน้า (สัญกรณ์ภาษาโปแลนด์) โดยมีอาร์กิวเมนต์ตามหลังตัวดำเนินการ สำหรับคำอธิบายทั้งหมดของไวยากรณ์ตัวกรองการค้นหา LDAP โปรดดู [RFC 4515](https://tools.ietf.org/html/rfc4515).
                                                                                                                                                                                                                                                                      
### LDAP Injection ทำงานอย่างไร?

เช่นเดียวกับ SQL Injection และการโจมตีด้วยการแทรกโค้ดที่เกี่ยวข้องช่องโหว่ของ LDAP Injection จะเกิดขึ้นเมื่อแอปพลิเคชันแทรกอินพุตของผู้ใช้ที่ไม่ได้รับการรับรองลงในคำสั่ง LDAP โดยตรง ด้วยการสร้างค่าสตริงที่เหมาะสมโดยใช้ไวยากรณ์ตัวกรอง LDAP ผู้โจมตีสามารถทำให้เซิร์ฟเวอร์ LDAP ดำเนินการค้นหาและคำสั่ง LDAP อื่น ๆ หากรวมกับสิทธิ์ที่กำหนดค่าไม่ถูกต้องหรือถูกบุกรุกการฉีด LDAP อาจทำให้ผู้โจมตีสามารถแก้ไขโครงสร้าง LDAP และแทรกแซงข้อมูลที่มีความสำคัญทางธุรกิจได้

ในขณะที่ LDAP Injection มีหลายรูปทรงและขนาดต่อไปนี้เป็นวิธีการทั่วไปสองวิธี:

- `Authentication bypass`: บริการไดเรกทอรีมักใช้สำหรับการตรวจสอบผู้ใช้และการอนุญาตดังนั้นการโจมตีด้วยการแทรก LDAP ขั้นพื้นฐานส่วนใหญ่จะพยายามข้ามการตรวจสอบรหัสผ่าน รับโค้ด JavaScript ที่มีช่องโหว่ต่อไปนี้ซึ่งประกอบโดยตรงกับตัวกรอง LDAP อย่างง่ายจากอินพุตของผู้ใช้ที่เก็บไว้ในตัวแปร `enterUser` และ `enterPwd`:

```bash
filterContent = "(&(userID=" + enteredUser + ")(password=" + 
  enteredPwd + "))"
```
สำหรับผู้ใช้ที่ไม่ประสงค์ร้ายตัวกรองผลลัพธ์ควรเป็นดังนี้: 

```bash
(&(userID=johndoe)(password=johnspwd))
```
หากแบบสอบถามนี้เป็นจริงชุดผู้ใช้และรหัสผ่านจะมีอยู่ในไดเร็กทอรีดังนั้นผู้ใช้จึงเข้าสู่ระบบอย่างไรก็ตามผู้โจมตีสามารถป้อนรหัสตัวกรอง LDAP เป็น ID ผู้ใช้ (แสดงเป็นสีแดง) เพื่อสร้างตัวกรองที่เป็นจริงเสมอ ตัวอย่างเช่น:

```bash
(&(userID=*)(userID=*))(|(userID=*)(password=anything))
```
สิ่งนี้สามารถทำให้ผู้โจมตีสามารถเข้าถึงได้โดยไม่ต้องมีชื่อผู้ใช้หรือรหัสผ่านที่ถูกต้อง

- `Information disclosure`: หากแอปพลิเคชันที่มีช่องโหว่ใช้ตัวกรอง LDAP เพื่อจัดเตรียมทรัพยากรที่ใช้ร่วมกันเช่นเครื่องพิมพ์ผู้โจมตีที่ทำการตรวจสอบใหม่อาจฉีดรหัสตัวกรอง LDAP เพื่อแสดงรายการทรัพยากรทั้งหมดในไดเรกทอรีขององค์กร สมมติว่าตัวกรองต่อไปนี้มีไว้เพื่อแสดงรายการเครื่องพิมพ์และเครื่องสแกนถูกประกอบในลักษณะที่ไม่ปลอดภัย:

```bash
(|(resType=printer)(resType=scanner))
```
หากผู้โจมตีสามารถป้อนค่าอื่นแทนเครื่องพิมพ์และทราบว่า userID ถูกใช้สำหรับชื่อผู้ใช้ในไดเร็กทอรีพวกเขาอาจแทรกโค้ดต่อไปนี้:

```bash
(|(resType=printer)(userID=*))(resType=scanner))
```
นี่จะแสดงรายการเครื่องพิมพ์และอ็อบเจ็กต์ผู้ใช้ทั้งหมดและส่วนของสแกนเนอร์จะถูกละเว้นโดยเซิร์ฟเวอร์ (เฉพาะตัวกรองที่สมบูรณ์ตัวแรกเท่านั้นที่ประมวลผล)

### Blind LDAP Injection

หากต้องการสอบถามเซิร์ฟเวอร์ LDAP โดยตรงผู้โจมตีจำเป็นต้องทราบ (หรือคาดเดา) ชื่อแอตทริบิวต์จึงจะระบุในตัวกรองได้ Blind LDAP injection เป็นเทคนิคการหาประโยชน์ขั้นสูงสำหรับการดึงข้อมูลที่ไม่รู้จักโดยการส่งคำขอหลายรายการและตรวจสอบการตอบกลับของเซิร์ฟเวอร์เพื่อตรวจสอบว่าแบบสอบถามนั้นถูกต้องหรือไม่ เมื่อรวมกับการเพิ่มประสิทธิภาพและระบบอัตโนมัติเพิ่มเติมทำให้ผู้โจมตีสามารถรับข้อมูลโดยใช้ชุดคำถามใช่ / ไม่ใช่: การตอบกลับของเซิร์ฟเวอร์ที่ถูกต้องหมายถึง“ ใช่” และข้อผิดพลาดของเซิร์ฟเวอร์หมายถึง“ ไม่ใช่” โดยทั่วไปแล้วการโจมตีด้วยการฉีดยาคนตาบอดที่มีประสิทธิภาพมักเกี่ยวข้องกับหลายขั้นตอน:

- `Attribute discovery`: ผู้โจมตีสามารถค้นหาคุณลักษณะที่เป็นไปได้หลายอย่างและตรวจสอบการตอบสนองของเซิร์ฟเวอร์ หากมีแอตทริบิวต์เซิร์ฟเวอร์จะตอบกลับที่ถูกต้อง มิฉะนั้นจะแสดงข้อผิดพลาดหรือการตอบกลับที่ว่าง สมมติว่าแอปพลิเคชันสร้างตัวกรอง AND เพื่อดึงข้อมูลผู้ใช้อย่างไม่ปลอดภัยเช่น:

```bash
(&(userID=John Doe)(objectClass=user))
```
หากผู้โจมตีสามารถปรับเปลี่ยนค่า ID ผู้ใช้พวกเขาสามารถ Inject โค้ดดังต่อไปนี้เพื่อตรวจสอบว่าออบเจ็กต์ผู้ใช้ในไดเร็กทอรีนี้มี department attribute:

```bash
(&(userID=John Doe)(department=*))(objectClass=user))
```
หาก `department` attribute มีอยู่ (และ `John Doe` เป็น ID ผู้ใช้ที่ถูกต้อง) เซิร์ฟเวอร์จะส่งคืนการตอบกลับที่ถูกต้อง มิฉะนั้นผู้โจมตีสามารถลองใช้ชื่อแอตทริบิวต์อื่น ๆ

- `Booleanization`: มื่อทราบชื่อแอตทริบิวต์แล้วผู้โจมตีสามารถส่งชุดคำขอที่มีสัญลักษณ์ตัวแทนและ / หรือตัวดำเนินการเปรียบเทียบเพื่อกำหนดค่าแอตทริบิวต์เฉพาะ อีกครั้งมีการพิจารณาการตอบสนองของเซิร์ฟเวอร์เพียงสองรายการดังนั้นการบูลีนจึงเป็นกระบวนการในการเปลี่ยนกระบวนการค้นหาเป็นการทดสอบจริง / เท็จหลายชุด

สมมติว่ามี `department` attribute จากตัวอย่างก่อนหน้านี้ ในการค้นหาชื่อแผนกผู้โจมตีสามารถเริ่มต้นด้วยการฉีดรหัสต่อไปนี้เพื่อตรวจสอบตัวอักษรตัวแรก:

```bash
(&(userID=John Doe)(department=a*))(objectClass=user))
```
การตอบกลับของเซิร์ฟเวอร์ที่ถูกต้องหมายความว่ามีแผนกที่ขึ้นต้นด้วยตัวอักษร“ a” อยู่ ผู้โจมตีสามารถดำเนินการต่อสำหรับ `ab*`, `ac*` และอื่น ๆ เพื่อค้นหาอักขระที่ตามมา สำหรับค่าตัวเลขสามารถใช้ตัวดำเนินการ <= (น้อยกว่าหรือเท่ากับ) และ> = (มากกว่าหรือเท่ากับ) เพื่อผ่านช่องว่างค่าที่เป็นไปได้

- `Character set reduction`: เพื่อลดจำนวนคำขอให้น้อยที่สุดผู้โจมตีสามารถใช้สัญลักษณ์แทนหลายตัวเพื่อค้นหาว่าอักขระใดอยู่ที่ใดก็ได้ในค่าเป้าหมาย ตัวอย่างเช่นการตอบสนองของเซิร์ฟเวอร์ที่ถูกต้องสำหรับการแทรกต่อไปนี้:

```bash
(&(userID=John Doe)(department=*x*))(objectClass=user))
```
หมายความว่ามีชื่อแผนกที่มีตัวอักษร“ x” อยู่ หากมีการส่งคืนข้อผิดพลาดหรือการตอบกลับที่ว่างเปล่าผู้โจมตีสามารถกำจัดอักขระนี้ออกจากการสแกนได้ สิ่งนี้สามารถลดจำนวนคำขอที่จำเป็นในการค้นหาค่าเป้าหมายได้อย่างมาก

### การป้องกัน LDAP Injection ในเว็บแอปพลิเคชัน

เช่นเดียวกับการโจมตีแบบฉีดอื่น ๆ การตรวจสอบความถูกต้องและการเข้ารหัสอินพุตที่เหมาะสมในชั้นแอปพลิเคชันเป็นสิ่งสำคัญในการกำจัดช่องโหว่ของการฉีด LDAP ทุกอินพุตของผู้ใช้ที่อาจใช้ในการสืบค้น LDAP ควรได้รับการฆ่าเชื้อตามข้อกำหนดของแอปพลิเคชันและเข้ารหัสเพื่อให้แน่ใจว่าอักขระพิเศษของ LDAP ที่เหลือจะถูกหลีกเลี่ยงอย่างปลอดภัยรวมถึงอย่างน้อย ()! | & *. เอกสารข้อมูลสรุป [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/LDAP_Injection_Prevention_Cheat_Sheet.html) มีข้อมูลโดยละเอียดเพิ่มเติมเกี่ยวกับเทคนิคการหลบหนี เพื่อความปลอดภัยและความสะดวกสูงสุดควรใช้เฟรมเวิร์กหรือไลบรารีที่พร้อมใช้สำหรับการหลีกเลี่ยงอักขระพิเศษและการประกอบตัวกรอง LDAP

### References:

- https://www.netsparker.com/blog/web-security/ldap-injection-how-to-prevent/


### Author
* Sanchat Phaisit
