## level 30

>bandit30:qp30ex3VLz5MDG1n91Yo************

There is a git repository at `ssh://bandit30-git@localhost/home/bandit30-git/repo` via the port 2220. The password for the user bandit30-git is the same as for the user bandit30.

Clone the repository and find the password for the next level.

### Solution:

```bash
# change to a /tmp directory
cd $(mktemp -d)

# clone the repo
git clone ssh://bandit30-git@localhost:2220/home/bandit30-git/repo

# cd into the repo and show tags
cd repo && git tag

# show the secret tag
git show secret
```