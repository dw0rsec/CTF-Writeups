## level 4

>bandit4:2WmrDFRmJIq3IPxneAaM************


The password for the next level is stored in the only human-readable file in the `inhere` directory. Tip: if your terminal is messed up, try the `reset` command.

### Solution:

```bash
# use the file command to locate the ASCII file in the inhere directory
for f in *; do file -- "$f"; done

# cat the file
cat -- '-file07'
```