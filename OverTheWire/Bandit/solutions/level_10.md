## level 10

>bandit10:FGUW5ilLVJrxX9kMYMml************

The password for the next level is stored in the file `data.txt`, which contains base64 encoded data.

### Solution:

```bash
# cat the file and pipe it to base64 to decode it
cat data.txt | base64 -d
```