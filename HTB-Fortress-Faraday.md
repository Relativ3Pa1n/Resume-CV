# HTB Fortress
## Faraday 
### Entrance 10.13.37.14 
```
└─$ ping 10.13.37.14                
PING 10.13.37.14 (10.13.37.14) 56(84) bytes of data.
64 bytes from 10.13.37.14: icmp_seq=1 ttl=63 time=76.2 ms
```
## Basic Nmap scan
```python
$nmap -v 10.13.37.14
Nmap scan report for 10.13.37.14
Host is up (0.082s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8888/tcp open  sun-answerbook

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 2.75 seconds
```
## Port 8888
```python
└─$ nc -nv 10.13.37.14 8888                                                  
(UNKNOWN) [10.13.37.14] 8888 (?) open
Welcome to FaradaySEC stats!!!
Username: root
Password: root
access denied!!!
```
 ## Port 22
 ```
 └─$ nc -nv 10.13.37.14 22                           
(UNKNOWN) [10.13.37.14] 22 (ssh) open
SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.2
```
## Port 80
![[Pasted image 20211212122918.png]]

Creating a login for the Alert system 

![[Pasted image 20211212123037.png]]

The Server isn/t configured.... We can create a SMTP server with python to see if it connects. Putting in some bogus info for now.
Getting past the config we start to get some information about the computers and Users:

![[Pasted image 20211212123849.png]]

![[Pasted image 20211212123750.png]]

We can start to collect this information for wordlists.

![[Pasted image 20211212124225.png]]

It looks like the site is set up to send alerts through smtp so we can set up a SMTP server and save any emails we get to a file. 
https://stackoverflow.com/questions/2690965/a-simple-smtp-server-in-python
```python
from datetime import datetime
import asyncore
from smtpd import SMTPServer

class EmlServer(SMTPServer):
    no = 0
    def process_message(self, peer, mailfrom, rcpttos, data, **kwargs):
        filename = '%s-%d.eml' % (datetime.now().strftime('%Y%m%d%H%M%S'),
            self.no)
        print(filename)
        f = open(filename, 'wb')
        f.write(data)
        f.close
        print('%s saved.' % filename)
        self.no += 1

def run():
    EmlServer(('localhost', 25), None)
    try:
        asyncore.loop()
    except KeyboardInterrupt:
        pass

if __name__ == '__main__':
    run()
```
Once our server is up we can try and get the web app to send us an email.
![[Pasted image 20211212141030.png]]

Back on our machine we can see that we have an email saved to a file and get the first flag.

```bash
┌──(kali㉿kali)-[~/htb/Fortress-Machines/Faraday/smtpServer]
└─$ ls                                                                             
20211212170740-0.eml  smtpserver.py    
┌──(kali㉿kali)-[~/htb/Fortress-Machines/Faraday/smtpServer]
└─$ cat 20211212170740-0.eml          
Subject: asdf

An event was reported at JohnConnor:
asdf
Here is your gift FARADAY{REDACTED}
```

