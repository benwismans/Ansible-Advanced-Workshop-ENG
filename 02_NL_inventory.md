## Lab 2: Inventory file aanmaken

An essential part of Ansible is the inventory file. This file contains the environment on which ansible code can be executed.

### Task 2.1: Inventory file aanmaken

In the inventory file are the hosts that can be reached with Ansible. An Ansible inventory works like a group, which can be put between square brackets
[ and ]. Within the group all clients are listed with either their DNS name or IP-address.  

* Edit de file inventory:

``$ vi inventory``

* Vul de inventory file met:

```
[loadbalancer]
lb ansible_host=localhost
[webservers]
web1 ansible_host=<hostname2>
web2 ansible_host=<hostname3>
```

## Task 2.2: telling Ansible where to find the inventory file
Ansible looks by default in the following path for the inventory file:

* /etc/ansible/hosts

In the configuration file ansible.cfg, an alternative path can be configured for the inventory file. Ansible looks by default in the following paths to an ansible.cfg file:

* ansible.cfg (in the current directory)
* .ansible.cfg (in the current directory
* /etc/ansible/ansible.cfg

By creating an ansible.cfg file in the same directory as the playbook (which we will create in a later lab), all default settings will be overruled by this ansible.cfg. We will point the inventory to  ~/inventory (the ~ is an alias for your home directory; the location where we created the inventory file. While we are it, we will also configure the user we use to login to the client. Use the username provided to you. We will disable host_key_checking.

* create the file ansible.cfg:
```
[defaults]
inventory = ~/inventory
remote_user = <user-on-client>
host_key_checking = False
```

### Task 2.3: Test if it works
Ansible works with modules. For each funtion, there is a module. For the creation of users we use the  ``user`` module. In our first steps we will use the ``ping`` module. This module makes it possible to check the connection from server to client. Different as you might be used to (from the ping command on the operating system), this module does not only check if your client is reachable (icmp reply) but also checks if Ansible can actually log on the the client with the provided credentials (using SSH by default on a Linux server). Also see: https://docs.ansible.com/ansible/latest/modules/ping_module.html#ping-module.

**Tip:** An overview of all modules can be found in the online documentation of Ansible: https://docs.ansible.com/ansible/latest/modules/list_of_all_modules.html.
* Check if the inventory file and the ansible.cfg file are in your home directory: 

``$ ls``


```
ansible.cfg  inventory
```
  
* Check if the Ansible module ``ping`` gives a correct answer:

``$ ansible --ask-pass -m ping webservers``


```
web1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
web2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

**Tip:** In our example, Ansible asks for an SSH password. In an automated scenario it's common to use with SSH Authorized Keys. In a later lab we will use this so Ansible can login directly, without password on the client. Because we do not have this set up yet, we give the ``--ask-pass`` instruction to ansible to ask for the SSH password.

In the inventoy file we created the group [webservers]. The ``ping`` module will therefor check all hosts in that group. 

Next step: [Lab 3 Playbook - User](03_NL_playbook_user.md)
