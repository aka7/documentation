###tunnel reverse: 
problem, from host 10.10.2.19 you want connect to 10.20.30.40:8901 but firewall is blocking it.
Solution:  if you have ssh access to 10.10.2.19, issue this command (reverse tunneling).  so you can access 10.20.30.40 on port 8901
```
ssh -R8901:10.20.30.40:8901 10.10.2.19
```
connecit to localhost 8901 will reverse tunnel to 10.20.30.40

###open tunnel. any port tunneling
if a jupmohst has access many service you can't get to it from your desktop, gui applications etc.
issue 

```
ssh -C2qTnN -D 127.0.0.1:8085 username@jumphost.domain
``
then on a browser, set socks proxy to localhost:8085

now you can access everytihng the jumphost is allowed to access via your browser.

## port forward, multiple

example
```
ssh -v user@host -L127.0.0.1:8443:10.10.0.66:443 -L127.0.0.1:5900:10.10.0.66:5900 -L127.0.0.1:5901:10.10.0.66:5901 -L127.0.0.1:5120:10.10.0.66:5120 -L127.0.0.1:5123:10.10.0.66:5123 -C
```
