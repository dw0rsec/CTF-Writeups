## level 2

>bandit2:263JGJPfgU6LtdEvgfWU************

The password for the next level is stored in a file called `--spaces in this filename--` located in the home directory

### Solution:

```bash
# You can use -- before the filename to indicate that what follows is not an option
cat -- "--spaces in this filename--"
```