## level 16

>bandit16:kSkvUpMQ7lBYyCM4GBPv************

The credentials for the next level can be retrieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. First find out which of these ports have a server listening on them. Then find out which of those speak SSL/TLS and which don’t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.

### Solution:

```bash
# check the password for the current level
cat /etc/bandit_pass/bandit16

# scan for open ports providing ssl with nmap
nmap -Pn -sV -vv -p 31000-32000 localhost

# use openssl to connect and paste in the password
openssl s_client -connect localhost:31790
```