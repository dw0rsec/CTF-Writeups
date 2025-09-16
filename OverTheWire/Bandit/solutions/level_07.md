## level 7

>bandit7:morbNTDkSW6jIlUc0ymO************

The password for the next level is stored in the file `data.txt` next to the word `millionth`.

### Solution:

```bash
# cat the file and use grep to filter the output
cat data.txt | grep -i 'millionth'
```