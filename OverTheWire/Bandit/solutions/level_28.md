## level 28

>bandit28:Yz9IpL0sBcCeuG7m9uQF************

There is a git repository at `ssh://bandit28-git@localhost/home/bandit28-git/repo` via the port 2220. The password for the user bandit28-git is the same as for the user bandit28.

Clone the repository and find the password for the next level.

### Solution:

```bash
# change to a /tmp directory
cd $(mktemp -d)

# clone the repo
git clone ssh://bandit28-git@localhost:2220/home/bandit28-git/repo

# cd into the repo and show commits
cd repo && git log

# inspect the "fix info leak" commit
git show 710c14a2e43cfd97041924403e00efb00b3a956e
```