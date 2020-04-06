# Git on Linux

### Commit without authenticating
Use SSH keys! 😍

1. Generate a SSH key like normal
```
# Generate an ssh key
ssh-keygen
```
2. Go to github -> settings -> keys and add your **public** key to your list of trusted keys.
3. Change the origin of the repo you are working on to use an ssh url rather than a conventional https url.

```
# Point your origin url for a git repo
git remote set-url origin git@github.com:<Username>/<Project>.git
```

```
# Point your origin url for a gist
git remote set-url origin git@gist.github.com:<Project code>
```

4. Verify that the url has changed
```
# Should change to the url you just set for the (fetch) and (push) links
git remote -v
# should output git@github.com:RolandWarburton/<Project>.git if SSH
```

### SSH commit Issue with VSCode
When you commit code with an ssh url you may get the following error
```ssh_askpass: exec(/usr/lib/ssh/ssh-askpass): no such file or directory```

To fix this you need to add that key to your authorization agent (openssh).
```ssh-add ~/.ssh/id_rsa```
