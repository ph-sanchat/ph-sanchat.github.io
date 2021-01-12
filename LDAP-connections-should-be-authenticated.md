# LDAP connections should be authenticated


การเชื่อมต่อ LDAP ควรได้รับการพิสูจน์ตัวตน
client LDAP พิสูจน์ตัวตนกับเซิร์ฟเวอร์ LDAP ด้วย "คำขอผูกมัด" ซึ่งมีวิธีการพิสูจน์ตัวตนแบบง่าย
### การพิสูจน์ตัวตนอย่างง่ายใน LDAP สามารถใช้ได้กับกลไกสามแบบ
* กลไกการพิสูจน์ตัวตนแบบไม่ระบุชื่อโดยดำเนินการร้องขอการผูกมัดด้วยชื่อผู้ใช้และรหัสผ่านที่สั้นๆ
* กลไกการพิสูจน์ตัวตนที่ไม่ได้รับการพิสูจน์ตัวตนโดยดำเนินการร้องขอการผูกมัดด้วยค่ารหัสผ่านที่สั้นๆ
* กลไกการตรวจสอบชื่อ / รหัสผ่านโดยดำเนินการร้องขอการผูกมัดด้วยค่ารหัสผ่านที่สั้นๆ

การผูกมัดแบบไม่ระบุชื่อและการโยงที่ไม่ได้รับการพิสูจน์ตัวตนอนุญาตให้เข้าถึงข้อมูลในไดเร็กทอรี LDAP โดยไม่ต้องระบุรหัสผ่านดังนั้นจึงไม่แนะนำให้ใช้

### ตัวอย่างรหัสที่ไม่เป็นไปตามข้อกำหนด
<pre>import ldap

def init_ldap():
   connect = ldap.initialize('ldap://example:1389')

   connect.simple_bind('cn=root') # Noncompliant
   connect.simple_bind_s('cn=root') # Noncompliant
   connect.bind_s('cn=root', None) # Noncompliant
   connect.bind('cn=root', None) # Noncompliant
</pre>

### โซลูชันที่รองรับ
<pre>import ldap
import os

def init_ldap():
   connect = ldap.initialize('ldap://example:1389')

   connect.simple_bind('cn=root', os.environ.get('LDAP_PASSWORD')) # Compliant
   connect.simple_bind_s('cn=root', os.environ.get('LDAP_PASSWORD')) # Compliant
   connect.bind_s('cn=root', os.environ.get('LDAP_PASSWORD')) # Compliant
   connect.bind('cn=root', os.environ.get('LDAP_PASSWORD')) # Compliant
</pre>

### Reference : https://rules.sonarsource.com/python/type/Vulnerability/RSPEC-4433

## Author
* Sanchat Phaisit
