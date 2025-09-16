## level 22

>bandit22:tRae0UfB9v0UzbCdn9cY************

A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in `/etc/cron.d/` for the configuration and see what command is being executed.

### Solution:

```bash
# change to the /etc/cron.d directory and cat the cronjob_bandit23 file
cd /etc/cron.d && cat cronjob_bandit23

# check the /usr/bin/cronjob_bandit23.sh file wich is running every minute

# run the command with user bandit23 instead of bandit22 to get the filename

echo I am user bandit23 | md5sum | cut -d ' ' -f 1

# cat the password
cat /tmp/8ca319486bfbbc3663ea0fbe81326349
```