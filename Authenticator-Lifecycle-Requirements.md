# Authenticator Lifecycle Requirements<h1>

| Preconditions | Inputs | Actions | Expect |
|-----------|-----------|-----------|-----------|
| Correct username <br>and password | Username <br>password | 1.Open page login<br>2.Add username <user><br>3.Add password <1234><br>4.click button login | 1.User can log in<br>2. Information page display system<br>3. Message system "Welcome to login" |
| Incorrect <br>password | Username <br>password | 1.Open page login<br>2.Add username <user><br>3.Add password <5555><br>4.click button login | 1.User cannot log in<br>2. Message system "Could not log in" please check username and password” |
| Incorrect | Username <br>password | 1.Open page login<br>2. click button login | 1. User cannot log in<br>2. Message system "Could not log in" please check username and password” |

### Team Author
* Wasupol Chaisangasilp
* Sanchat phaisit
