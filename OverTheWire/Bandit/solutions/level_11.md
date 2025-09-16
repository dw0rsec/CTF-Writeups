## level 11

>bandit11:dtR173fZKb0RRsDFSGsg************

The password for the next level is stored in the file `data.txt`, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions.

### Solution:

I wrote a python script to rot13 decode the string in `data.txt`.

```python
import string

upper = list(string.ascii_uppercase)
lower = list(string.ascii_lowercase)

def rot13(encoded_password):
    decoded_password = ''
    for char in encoded_password:
        if char in upper:
            decoded_password += upper[(upper.index(char) + 13) % 26]
        elif char in lower:
            decoded_password += lower[(lower.index(char) + 13) % 26]
        else:
            decoded_password += char
    return decoded_password

if __name__ == '__main__':
    print(rot13('Gur cnffjbeq vf 7k16JArUVv5LxVuJfsSVdbbtaHGlw9D4'))
```