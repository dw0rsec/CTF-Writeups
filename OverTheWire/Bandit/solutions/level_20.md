## level 20

>bandit20:0qXahG8ZjOVMN9Ghs7iO************

There is a setuid binary in the homedirectory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).

### Solution:

```bash
# echo the password of the previous level and pipe it into a netcat listener
echo -n '0qXahG8ZjOVMN9Ghs7iO************' | nc -l -p 4444 &

# use the suid binary to connect to the netcat listener
./suconnect 4444
```