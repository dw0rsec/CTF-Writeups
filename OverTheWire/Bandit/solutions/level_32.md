## level 32

>bandit32:3O9RfhqyAlVBEZpVb6LY************

After all this git stuff, it’s time for another escape. Good luck!

### Solution:

```bash
# break out of the uppercase shell
$0

# check which user we are running
id

# since our uid is bandit33 we can grab the password
cat /etc/bandit_pass/bandit33
```