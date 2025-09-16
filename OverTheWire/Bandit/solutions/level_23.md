## level 23

>bandit23:0Zf11ioIjMVN551jX3Cm************

A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.

### Solution:

```bash
# change to the /etc/cron.d directory and cat the cronjob_bandit24 file
cd /etc/cron.d && cat cronjob_bandit24

# check the /usr/bin/cronjob_bandit24.sh file wich is running every minute

# change to the /tmp directory 
cd $(mktemp -d)

# create following script

#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/tmp.jYGnfvoM7U/password

# create the password file
touch password

# change permissions

chmod 777 password && chmod 777 getpass.sh

# copy it to /var/spool/bandit24/foo/
cp getpass.sh /var/spool/bandit24/foo/

# grab the password from the password file
cat password
```