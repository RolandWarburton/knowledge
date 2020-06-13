# Github

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
