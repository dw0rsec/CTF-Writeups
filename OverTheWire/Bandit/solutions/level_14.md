## level 14

>bandit14 sshkey.private

The password for the next level can be retrieved by submitting the password of the current level to port 30000 on localhost.

### Solution:

```bash
# check the password for the current level
cat /etc/bandit_pass/bandit14

# use netcat to conect to localhost on port 30000 and paste the password
nc localhost 30000
```