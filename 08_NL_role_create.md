## Lab 8: Role - Build One Yourself

In Ansible it is best-practice to work with roles. It's good to know how you can build your own role. We will do that in this lab to build a ``chrony`` role to configure chrony.

NOTE: In RHEL 7+ / CentOS 7+  chrony is the standaard daemon for the network time protocol (NTP). 

A role exists of the following parts (directories)

* ``tasks``: All tasks for de role.
* ``handlers``: Contains halders that are run with ``notify``. It's used a lot to restart services after a change.. See https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#task-and-handler-organization-for-a-role for details.
* ``defaults``: Default variables for the role. These have the lowest priority, so they are easy to overrule.
* ``vars``: Variables voor de role. Variables in ``vars`` have a higher priority than variables in ``defaults``.
* ``files``: Contains files used for the role. These can be used with the ``copy`` module.
* ``templates``: Contains files with variables. For templates the Jinja2 format is used as templating code. 
* ``meta``: Contains metadata for the role, like: Author, supporting platforms of dependencies with other roles.

The directories ``tasks``, ``handlers``, ``defaults``, ``vars`` and ``meta`` ned to contain a file ``main.yml``. That file contains the actual code of every part. Every ``main.yml`` starts with ``---``. Even if there is nothing else in the file.

```
---
```

### Task 8.1: Creating an empty role

With the command ``ansible-galaxy init --offline <rolename>`` you can create a skeleton for your new role. 

NOTE: Roles will be place by default in ``/etc/ansible/roles`` or ``~/.ansible/roles/`` (in your home directory).

* Create a new role :

```
$ cd ~/.ansible/roles/
$ ansible-galaxy init --offline chrony
```

* With ``find`` you can get an overview of the files in your skeleton directory:
```
$ find chrony
chrony
chrony/README.md
chrony/defaults
chrony/defaults/main.yml
chrony/files
chrony/handlers
chrony/handlers/main.yml
chrony/meta
chrony/meta/main.yml
chrony/tasks
chrony/tasks/main.yml
chrony/templates
chrony/tests
chrony/tests/inventory
chrony/tests/test.yml
chrony/vars
chrony/vars/main.yml
```

### Task 8.2: Handler file

In the file ``handlers/main.yml`` you can define the handler. A ``handler`` is only run when a task has a change. 

* Add to the file ``handlers/main.yml``:
```
---
- name: Restart chrony
  systemd:
    name: chronyd
    state: restarted
```
**TIP:** This handler restarts ``chronyd`` when the taks has a change.  

### Task 8.3: Default variables 

The default variables are in  ``defaults/main.yml`` gezet. If you install a service and configure it, it is common to put that name in a default variable so you can reuse it again.

* Add the following  in the file ``defaults/main.yml``:
```
---
chrony_package: chrony
chrony_service: chronyd
```

### Task 8.4: Config file 

* Creat a new file ``files/chrony.conf``:
```
# Use these NTP servers:
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Specify directory for log files.
logdir /var/log/chrony
```

### Task 8.5: Tasks

The last step is to  add the tasks. This is comparible with the ``tasks`` in a ``playbook``.

* Adjust the file ``tasks/main.yml``:
```
---
- name: Ensure chrony is installed
  yum:
    name: chrony
    state: latest

- name: Ensure chrony is configured
  copy:
    src: chrony.conf
    dest: /etc/chrony.conf
  notify: Restart chrony

- name: Ensure chrony is enabled and started
  systemd:
    name: chronyd
    enabled: yes
```

**TIP:** This line ``notify: Restart chrony`` makes sure the right handler is run after a change.

### Task 8.6: Playbook

* Create a new playbook ``chrony.yml`` to run the role. Look back in link:06_NL_role_haproxy[Lab 6 Role - HAProxy] how to create a playbook with a role.

```
$ cd ~
```
* Run the playbook 

**NOTE:** The playbook will have a change on the ``task`` ``Ensure chrony is configured``. Therefor the ``Restart chrony`` will be executed. The handler is always run at the end of the playbook.


Next Step: [Lab 9 Role - Templates ](09_NL_templates.md)
