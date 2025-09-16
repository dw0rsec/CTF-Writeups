## level 21

>bandit21:EeoULMCra2q0dSkYj561************

A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in `/etc/cron.d/` for the configuration and see what command is being executed.

### Solution:

```bash
# change to the /etc/cron.d directory and cat the cronjob_bandit22 file
cd /etc/cron.d && cat cronjob_bandit22

# check the /usr/bin/cronjob_bandit22.sh file wich is running every minute

# the script runs cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv so you can grab the password there
cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```