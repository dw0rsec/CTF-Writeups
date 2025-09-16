## level 12

>bandit12:7x16WNeHIi5YkIhWsfFI************

The password for the next level is stored in the file data.txt, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under `/tmp` in which you can work. Use mkdir with a hard to guess directory name. Or better, use the command `mktemp -d`. Then copy the datafile using cp, and rename it using mv (read the manpages!).

### Solution:

```bash
# revert the hexdump back with xxd
cat data.txt | xxd -r > data

# check the file type
file data

# rename the file and decompress it with gzip
mv data data.gz && gzip -d data.gz

# check the file type of the decompressed file
file data

# rename the file and decompress it with bzip2
mv data data.bz2 && bzip2 -d data.bz2

# again check the file type of the decompressed file
file data

# rename the file and decompress it with gzip
mv data data.gz && bzip2 -d data.gz

# repeat until you get data8 as a ascii file and cat it
cat data8
```