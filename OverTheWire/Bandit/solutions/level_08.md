## level 8

>bandit8:dfwvzFQi4mU0wfNbFOe9************

The password for the next level is stored in the file `data.txt` and is the only line of text that occurs only once.

### Solution:

```bash
# cat the file and pipe it to sort and uniq
cat data.txt | sort | uniq -u
```