## level 18

>bandit18:x2gLTTjFwMOhQ8oWNbMN************

The password for the next level is stored in a file `readme` in the homedirectory. Unfortunately, someone has modified .bashrc to log you out when you log in with SSH.

### Solution:

```bash
# just specify /bin/sh as a shell when connecting over ssh
ssh bandit18@bandit.labs.overthewire.org -p 2220 -t /bin/sh
```