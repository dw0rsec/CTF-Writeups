## level 29

>bandit29:4pT1t5DENaYuqnqvadYs************

There is a git repository at `ssh://bandit29-git@localhost/home/bandit29-git/repo` via the port 2220. The password for the user bandit29-git is the same as for the user bandit29.

Clone the repository and find the password for the next level.

### Solution:

```bash
# change to a /tmp directory
cd $(mktemp -d)

# clone the repo
git clone ssh://bandit29-git@localhost:2220/home/bandit29-git/repo

# cd into the repo and show branches
cd repo && git branch -a

# change to the dev branch to grab the password
git checkout dev
```