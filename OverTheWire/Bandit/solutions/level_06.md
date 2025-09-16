## level 6

>bandit6:HWasnPhtq9AVKe0dmk45************

The password for the next level is stored somewhere on the server and has all of the following properties:

- owned by user bandit7
- owned by group bandit6
- 33 bytes in size

### Solution:

```bash
# use the find command to locate the file with the specs mentioned above
find / -type f -size 33c -user bandit7 -group bandit6 2>/dev/null

# cat the file
/var/lib/dpkg/info/bandit7.password
```