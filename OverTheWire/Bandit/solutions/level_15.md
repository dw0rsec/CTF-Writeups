## level 15

>bandit15:8xCjnmgoKbGLhHFAZlGE************

The password for the next level can be retrieved by submitting the password of the current level to port 30001 on localhost using SSL/TLS encryption.

### Solution:

```bash
# check the password for the current level
cat /etc/bandit_pass/bandit15

# use openssl to connect and paste in the password
openssl s_client -connect localhost:30001
```