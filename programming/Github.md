# Github

## Git commands

### Merge Two Repos Into One

Beware this will not create a subfolder for RepoB in RepoA, it will copy everything over to the base die

If you want to merge project-b into project-a:

```bash
cd path/to/project-a
git remote add project-a path/to/project-b
git fetch project-b --tags
git merge --allow-unrelated-histories project-b/master
git remote remove project-b
```

### Enable SSH for repo

I have instructions for how to do this on linux [here](https://rolandw.dev/Notes/Linux/GitonLinux/), i have not tried it on windows so YMMV.

### Untracking a file

In the situation that you originally were tracking/committed a file and no longer want to track it (a common example is .env) you need to tell github to untrack it, as well as add it to the .gitignore file.

Run this command to cause github to "forget" about the file. Specify the filepath from the .git folder.

```none
# Untrack the file in app/.env
git rm --cached app/.env
```

Then add it to the `.gitignore`.

```none
.env
# or
app/.env
# or
*.env
```
