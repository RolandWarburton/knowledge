# Ansible

## Installing

```none
apt install python3 python3-pip
pyrhon3 -p pip install --user ansible
```

Ansible is installed to `$HOME/.local/bin/ansible`.

Auto complete can be achieved in zsh with the following.

```none
autoload -U bashcompinit
bashcompinit

eval $(register-python-argcomplete ansible)
eval $(register-python-argcomplete ansible-config)
eval $(register-python-argcomplete ansible-console)
eval $(register-python-argcomplete ansible-doc)
eval $(register-python-argcomplete ansible-galaxy)
eval $(register-python-argcomplete ansible-inventory)
eval $(register-python-argcomplete ansible-playbook)
eval $(register-python-argcomplete ansible-pull)
eval $(register-python-argcomplete ansible-vault)
```

## Simple Setup

Create the following as your hosts file.

The hosts file contains a list of machines you control through ansible.

```ini
<!-- inventory/hosts.ini -->
[hosts]
192.168.1.102
192.168.1.103
```

You should also install `sshpass`. Non-interactive ssh password authentication.

```none
sudo apt install sshpass
```

### Ping Module

Lets use the inbuilt ping module to ping each machine.

```none
ansible \
  -i ./inventory/hosts.ini 192.168.1.102 \
  -m ping \
  --user roland \
  --ask-pass
```

### Apt module

Create a playbook that uses the apt module to `update` and `upgrade` debian machines.

```yml
- hosts: "*"
  become: yes # become root
  tasks:
    - name: apt
      apt:
        update_cache: yes
        upgrade: yes
```

Here is an example running the playbook on a single host.

```none
ansible-playbook \
  --limit 192.168.1.102 ./playbooks/apt.yml \
  --user roland \
  --ask-pass \
  --ask-become-pass \
  -i ./inventory/hosts.ini
```

### Using SSH Keys

On the ansible control host.

```none
ssh-keygen
ssh-copy-id ~/.ssh/id_rsa 192.168.1.102
ssh 192.168.1.102
```

```none
ansible all \
  --key-file ~/.ssh/id_rsa \
  -i inventory/hosts.ini --limit 192.168.1.102\
  -m ping
```

### Creating an Ansible Config

```ini
<!-- ansible.cfg -->
[defaults]
inventory·=·./inventory/hosts.ini
private_key_file·=·~/.ssh/id_rsa
```

The ansible config will be picked up when run from the same directory.

You can learn more about limiting hosts to run on 
[here](https://docs.ansible.com/ansible/latest/user_guide/intro_patterns.html).

```none
# ping all hosts
ansible all -m ping

# ping one host
ansible 192.168.1.102 -m ping
```

### Ansible Toolbox

Get all ansible hosts.

```none
ansible all --list-hosts
```

Get all facts about ansible hosts

```none
ansible all -m gather_facts
ansible all -m gather_facts --limit 192.168.1.102
```

### Ad-Hoc Commands With Root

```none
ansible all \
  -m apt \
  -a update_cache=true \
  --become \
  --ask-become-pass
```

Installing a package with apt.

```none
ansible all \
  -m apt \
  -a name=nano \
  --become --ask-become-pass
```

upgrading a package with apt.

```none
ansible all \
  -m apt \
  -a "name=nano state=latest" \
  --become --ask-become-pass
```

Upgrade everything with apt.

```none
ansible all \
  -m apt \
  -a "update_cache=true upgrade=dist" \
  --become --ask-become-pass
```

Install a package with apt.

```none
ansible all \
  -m apt \
  -a "name=apache2" \
  --become --ask-become-pass
```

We can convert this into a playbook however because we would want to update the apt cache first.

```yaml
# install-apache.yml
- hosts: "*"
  become: yes # become root
  tasks:
    - name: update apt cache
      apt:
        update_cache: yes
    - name: install apache2
      apt:
        name: apache2
```

Ensure that you have the above `ansible.cfg` file. Then run the playbook.

```none
ansible-playbook \
  --limit 192.168.1.102 ./playbooks/install-apache.yml \
  --ask-pass --ask-become-pass \
```

### Print Commands Results to stdout

Ansible can execute arbitrary commands using the `ansible.builtin.shell` module.

```yaml
# print-to-stdout.yml
- hosts: 192.168.1.102
  tasks:
    - name: echo whoami
      ansible.builtin.shell: "whoami"
      register: hello
    - debug: msg="{{ hello.stdout }}"
```

```none
ansible-playbook \
  ./playbooks/ping.yml \
```

### Ping Example Playbook

```yaml
# print-to-stdout.yml
- hosts: 192.168.1.102
  tasks:
    - name: ping stuff
      ping:
```

### When Conditional

you can determine different ansible variables such as `ansible_distribution`.

```yaml
- hosts: "*"
  become: yes
  tasks:
    - name: update apt cache
      when: ansible_distribution == "Debian" # Add this
      apt:
        update_cache: yes
    - name: install apache2
      # when: ansible_distribution in ["Debian", "Ubuntu"] # Add this (alternative syntax)
      when: ansible_distribution in ["Debian", "Ubuntu"] and ansible_apparmor.status == "disabled"
      apt:
        name: apache2
```

If you want to chain things together you can also write.

```yaml
when: ansible_distribution in ["Debian", "Ubuntu"] and ansible_apparmor.status == "disabled"
```

When conditionals can also look at task state from `registers` in other tasks.
For example to determine if you should restart a service if it has changed or not.

```yaml
- name: write a file
  copy:
    src: /home/{{ansible_user_id}}/ansible/file.txt
    dest: /home/{{ansible_user_id}}/file.txt
    owner: roland
  register: fileCopy
- name: restart my service
  service:
    name: "myservice"
    state: restarted
  when: fileCopy.changed
```

### Variables

The below playbook uses `message` as a variable.

It also doubles as a good way to debug output and pass state between plays.

```yaml
# say-a-word.yml
- hosts: 192.168.1.102
  tasks:
    - name: say something
      shell: "echo whoami"
      register: hello
    - debug: msg="{{ hello.stdout }}"
```

Add `message` to the inventory file.

```ini
[hosts]
192.168.1.102 message=hello
192.168.1.103
192.168.1.104
192.168.1.105
```

Another way of creating variables can be done inside the playbook.

```yaml
- hosts: 192.168.1.102
  tasks:
    - name: say something
      shell: "echo whoami"
      register: hello
```

### Tags

Tags can be defined to make running specific plays easier in a playbook.

```yaml
- hosts: 192.168.1.102
  tags: tag4
  tasks:
    - name: ping a computer always
      tags: always # will always run
      ping:
    - name: ping a computer
      tags: tag3
      ping:
    - name: ping a computer again
      tags: tag1, tag2
      ping:
```

List tags with.

```none
ansible --list-tags ./playbooks/say-a-word.yml
```

Run plays in playbook with specific tags

```none
ansible-playbook \
  ./playbooks/say-a-word.yml \
  --tags "tag1,tag2"
```

### Managing Files

Copy file to node.

```yaml
- hosts: "192.168.1.102"
  tasks:
    - name: copy file
      copy:
        src: /home/{{ansible_user_id}}/ansible/file.txt
        dest: /home/{{ansible_user_id}}/file.txt
        group: {{ansible_user_id}}
        owner: {{ansible_user_id}}
        mode: 0755
```

The other way to get the users name is to use the following play and store it in a register.

```yaml
- hosts: "192.168.1.102"
  vars:
    a: myvar
  tasks:
    - name: get the username running the deploy
      become: false
      local_action: command whoami
      register: username
```

### Services

```yaml
- hosts: "192.168.1.102"
  tags: runme
  become: yes
  tasks:
    - name: install nginx
      when: ansible_distribution == "Debian"
      apt:
        name: nginx
    - name: start a service
      when: ansible_distribution == "Debian"
      tags: myservice
      service:
        name: nginx.service
        state: started
        enabled: yes
```

### Change a line in a file

```yaml
- hosts: "192.168.1.102"
  tasks:
    - name: install nginx
      when: ansible_distribution == "Debian"
      apt:
        name: nginx
    - name: change line in file
      when: ansible_distribution == "Debian"
      tags: myservice
      register: result # for debugging purposes
      lineinfile:
        path: /home/{{ansible_user_id}}/file.txt
        regexp: "^two"
        line: "number 2"
    - debug: msg="{{ result }}"
```

### Bootstrapping

Create a user.

```yaml
- hosts: "192.168.1.102"
  become: true
  tasks:
    - name: create a user "rinne"
      user:
        name: rinne
        groups: root
    - name: add ssh key for rinne
      authorized_key:
        user: rinne
        key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC3W0kYY1Ej/DQOOkZrcZc8HhIktKSDfBKJa2RIH6xIC4n4lpJKGtnZ4rb2/8Q76eXceNNcqalxc0nJhIQtQVxM5+VjQeXnPGN6kpWJp6n9qcVLYXsRwtnLRJiVr8RfmcbRgjiGsRdXuG7jzi4xpPFnPJ7I07EJwXVu6bA68glTtYXGSbfXtX2OWbhUaRf3j1QnwtzAVz0+CLkPAvaeQisBLMChLsEux3QZHJWPvezB3dUMtOyhlqMVCjTTRVOvmqx70IHktF4hNsojNs67eLCApwjLmxIwcC2lXt2N+Xgi9K9OenaofBhZ5xjd8lw+w7s0xZ+zII0ywRPWSI6UmMRL+h4vR3bbbOhDcFk1Ga06l6KhNxzULCYCkEslgUiBkz7LeB/FQdjqA+DfgkxFHEDODn4eyRgFZRZCi1cuPGYEJb7ptcBgPLS8K3cpkQziYQNUa+zcUJBcNAN6HWODFfxiZ2NVXMqDd89VES/fvO8jXrb4SkVXc4907ldtMhL/6Sc= roland@DEB01"
    - name: create sudoers file for rinne
      copy:
        src: sudoer_rinne
        dest: /etc/sudoers.d/rinne
        owner: root
        group: root
        mode: 0440
```

Create the `sudoers_rinne` file too in `./playbooks/files/sudoer_rinne`.

```none
rinne ALL=(ALL) NOPASSWD: ALL
```


### Avoid passing ask-become-pass

Now that we have a user that is in the sudoers NOPASSWD group we can omit the `--ask-become-pass`.

Change the `ansible.cfg` file to be like so.

```ini
<!-- ansible.cfg -->
[defaults]
inventory = ./inventory/hosts.ini
private_key_file = ~/.ssh/id_rsa
remote_user = rinne
```

### Roles

Tasks can be reduced into roles of "task books" that can be assigned to "playbooks".

A task book stored in `./roles/<task name>/tasks/main.yml` contains a list of tasks that will
be performed.

First create a playbook with a role. We will then create a role called "base".

```yaml
# ./playbooks/roles.yml
- hosts: "192.168.1.102"
  become: true
  vars:
    username: roland # we will need this to perform actions on the user as we are running as root
    filename: myfile # will make sense when demonstrating how files are located in tasks
  roles:
    - ./roles/base
```

Create the base role.

```yaml
# ./roles/base/tasks/main.yml
- name: add ssh key for rinne
  authorized_key:
    user: rinne
    key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC3W0kYY1Ej.."

- name: copy file
  copy:
    src: file.txt
    dest: /home/{{username}}/{{filename}}.txt
    owner: roland
```

You can then run this playbook as usual.

```none
ansible-playbook \
  ./playbooks/roles.yml
```

### Host Variables

You can define variables that are available to a host in any context (play book, task book, etc).

Place a `host_vars` folder in the same directory as `inventory.ini`.

```yaml
# ./inventory/host_vars/192.168.1.102
username: roland
filename: myfile
```

You can then modify your playbook and inventory file to remove all variable mentions.

```ini
<!-- ./inventory/hosts.ini -->
[workstations]
192.168.1.102 remove_me=yes
```

```yaml
# ./playbooks/roles.yml (can be any playbook)
- hosts: workstations
  become: true
  roles:
    - ./roles/base
```

### Notify Task Trigger

Instead of using a changed event on a task to determine if a task has run or not,
you can use notify.

We will replace the below task with a new one.

```yaml
- name: write a file
  copy:
    src: /home/{{ansible_user_id}}/ansible/file.txt
    dest: /home/{{ansible_user_id}}/file.txt
    owner: roland
  register: fileCopy
- name: restart my service
  service:
    name: "myservice"
    state: restarted
  when: fileCopy.changed
```

```yaml
./roles/<role name>/tasks/main.yml
- name: write a file
  copy:
    src: /home/{{ansible_user_id}}/ansible/file.txt
    dest: /home/{{ansible_user_id}}/file.txt
    owner: roland
  notify: restart_myservice # Change this
  # delete the task to restart the service
```

Next create a new handlers folder and file in your roles directory.

```yaml
# ./roles/base/handlers/main.yml
- name: restart_myservice # named the same as the "notify"
  service:
    name: "myservice"
    state: restarted
```

### Templates

Templates allow for you to inject many different variables
from a host_vars file into a single config.

Create a new folder `./roles/base/templates`.

```none
# ./roles/base/templates/config.j2
AllowUsers {{ workstation_users }}
```

```yaml
# ./inventory/host_vars/192.168.1.102
workstation_users: "roland rinne"
workstation_template_file: "config.j2"
```

```yaml
# ./roles/tasks/main.yml
- name: generate file from template
  template:
    src: "{{ workstation_template_file }}"
    dest: /home/{{username}}/config
    owner: "{{username}}"
    mode: 755
  notify: restart_sshd
```

```yaml
# ./roles/base/handlers/main.yml
- name: restart_sshd
  service:
    name: sshd
    state: restarted
```
