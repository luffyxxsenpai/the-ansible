# ansible-day1

### What is Ansible?
- Ansible is an open-source IT automation tool.
- It simplifies configuration management, application deployment, and orchestration.
- Ansible uses a simple YAML-based syntax called Playbooks.
- It is agentless, meaning no software needs to be installed on managed nodes.
- Ansible operates over SSH, making it lightweight and secure.

### How Ansible Works
- Control Node:
    - The machine where Ansible is installed and run.
    - Executes tasks on managed nodes.

- Managed Nodes:
    - The servers or devices that Ansible controls.
    - Requires only SSH access or Windows Remote Management (WinRM) for Windows nodes.

- Inventory:
    - A file listing the servers to be managed.
    - Supports grouping of hosts and variable assignments.

- Modules:
    - Predefined tools to perform specific tasks (e.g., installing packages, copying files, restarting services).
    - Modules are executed on managed nodes.

- Playbooks:
    - YAML files defining a set of tasks to automate processes.
    - Organized in a sequential manner.

### Idempotency:
- idempotency means that running the same task multiple times will not make unnecessary changes if the desired state is already achieved. 
- This ensures tasks are only applied if needed, making automation predictable and safe.
- lets say we defined a playbook to install nginx
    - in first run, it will install nginx
    - now in 2nd time, if nginx is already present, the task will be skipped as the nginx is already in desired state


# INVENTORY FILE
- inventory file is used to define the hosts or servers we are gonna manage
- it can be written in ini or yaml format
- inventory files can be of 2 types
    - static inventory file is manually managed means admin have to manually define the host address and other parameters.
    - dynamic inventory file is used to automatically create the inventory file when we are working or scalable infra.
 
- there is a default inventory file at /etc/ansible/hosts but we create our own custom inventory file for more control
- we can name it anything.ini or anything.yaml and will use `-i filename` to use the specific inventory file


**use this to ping the server specified in inventory file `ansible all -i inventory.ini -m ping` **
** we still haven't set up the authentication yet so we will see error while pinging but it still tells us which server its trying to ping**
### simplest inventory file
- just dump all the server ip in the file
```ini
myserver.aws.com
myserver2.aws.com
1.2.3.4
5.6.7.8

```
`ansible all -i inventory.ini -m ping` will ping all the mentioned servers

### grouped inventory file
- we define groups by `[group-name]` 
```ini
[frontend]
1.2.3.4
5.6.7.8

[backend]
2.3.4.5
6.7.8.9

[test]
5.6.7.8
```
`ansible all -i inventory.ini -m ping` will ping all the groups 
`ansible frontend -i inventory.ini -m ping` will only ping the frontend
`ansible backend -i inventory.ini -m ping` will only ping the backend

### variables
- we can use various variables to make our inventory a bit modular and provide authenticaiton parameters
- `server1 ansible_host=1.2.3.4` can name our servers but will need to use ansible_host to define the ip
- `ansible_user=ubuntu` to user ansible logs in as
- `ansible_password or ansible_pass or ansible_ssh_password` to define ssh password ansible will be using, requires `sshpass` package
- `ansible_ssh_private_key_file=/path/to/key` to use the pem file required for authentication
- `ansible_ssh_port` to define the ssh port, don't need if we are using the default port 22
- `ansible_ssh_retries=1` number of attempts to connect
- `ansible_ssh_timeout=15` amount of time to wait while establishing the connection
- we can define these variable in another block using `[groupname:vars]`
- `[newgrp:children]` creates group of group 

- refer to docs for other variables

```ini
[frontend]
server1 ansible_host=1.2.3.4
server2 ansible_host=5.6.7.8

[backend]
server3 ansible_host=2.3.4.5
server4 ansible_host=6.7.8.9

[test]
server5

[all:vars]
ansible_user=ubuntu

[frontend:vars]
ansible_ssh_private_key_file=/home/myuser/secretkey.pem

[solo:vars]
ansible_host=4.5.6.7
ansible_ssh_password=mysecretpass

[webserver:children]
frontend
backend

```
- now this is a quite good inventory file

---
## enable password auth in ec2
- `sudo passwd ubuntu` create password for ubuntu user
- we can either directly edit the `/etc/ssh/sshd_confing` and edit the line `PasswordAuthentication` to yes.
- or we can edit this custom file ec2 has to enable the password base authentication 
- `sudo vi /etc/ssh/sshd.config.d/<40_custom_something>` and change it to yes
- `sudo systemctl restart ssh` to restart the ssh service to reflect the changes
- now use the password to login, disable root login 
- we did this for solo group server

## keyless authentication in ec2
- the goal is to create a pair of key and copy the public key we created to the ec2 servers in which we want keyless authentication
- next time we wont have to give any password or specify pem file to login
- `ssh-keygen` to create a pair of keys, will be in ~/.ssh by default
- `ssh-copy-id -f "-o IdentityFile /path/to/file.pem" ubuntu@3.1.2.4` will copy our public key in server's `~/.ssh/authorized_key` file
- we did this for backend server


---

