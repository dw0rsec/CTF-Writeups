## level 31

>bandit31:fb5S2xb7bRyFmAvQYQGE************

There is a git repository at `ssh://bandit31-git@localhost/home/bandit31-git/repo` via the port 2220. The password for the user bandit31-git is the same as for the user bandit31.

Clone the repository and find the password for the next level.

### Solution:

```bash
# change to a /tmp directory
cd $(mktemp -d)

# clone the repo
git clone ssh://bandit31-git@localhost:2220/home/bandit31-git/repo

# cd into the repo and read the README.md
cd repo && cat README.md

# create the key.txt file
echo 'May I come in?' > key.txt

# add the file with git add to circumvent .gitignore
git add -f key.txt

# commit your add
git commit -a

# push your changes
git push -u origin master
```