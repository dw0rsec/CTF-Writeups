## level 9

>bandit9:4CKMh1JI91bUIZZPXDqG************

The password for the next level is stored in the file `data.txt` in one of the few human-readable strings, preceded by several `=` characters.

### Solution:

```bash
# use the strings command and grep for the =
strings data.txt | grep '=='
```