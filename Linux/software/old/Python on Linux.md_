# PIP and Python

### Environments

> "aughhhh pip is a mess! I dont want pip packages installed globally!!!"\
Create an environment for pip stuff in a folder (works with python3 only)

Install venv using `apt`. On Arch install any of these packages [source](https://wiki.archlinux.org/index.php/Python/Virtual_environment).

* Python 3.3+: python
* Python 3: python-virtualenv

```none
# Debian install
sudo apt install python3-venv
```

Create a venv in the current directory

```none
python3 -m venv .
```

Activate the venv

```none
source bin/activate
```

Once you are finished you can exit by running `deactivate`.

### PIP packages are not recognized

Install the python extension to python and use the bottom left menu titled 'python 3.x.x 64 bit' and click on it to change the environment.

### Cant import package thats nested in a folder

use the syntax `import foldername.somefile`.

### pip list Vs pip freeze

without installing any extra packages there are 43 packages in pip list and 39 packages in pip freeze.

* pip list is longer because it has the package *setuptools* and *pip* itself.
* Pip freeze is used to generate the *requirements.txt* file for automatically installing a list of packages.
