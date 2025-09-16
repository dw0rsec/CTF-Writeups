## level 5

>bandit5:4oQYVPkxZOOEOO5pTW81************

The password for the next level is stored in a file somewhere under the `inhere` directory and has all of the following properties:

- human-readable
- 1033 bytes in size
- not executable

### Solution:

```bash
# use the find command to locate the file with a size of 1033 bytes
find . -type f -size 1033c

# cat the file
cat ./maybehere07/.file2
```