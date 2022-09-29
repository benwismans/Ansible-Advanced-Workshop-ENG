## Lab 3: Playbook - User configuration

A playbook is a description of how a system should be configured. This playbook contains a list of steps. Every step checks the current condition and adjusts it, if necessary. If the system already matches the description of the step, Ansible does nothing. We call this idempotent.

**Tip:** Try to create your playbook so no changes are made anymore, when the playbook is run for the 2nd time. It's usual to run a playbook multiple times without giving any issues.

In this lab we create a playbook for installing the public key for SSH. After the installation of the public key, you can logon with your private key (without password) on the client system.

**Tip:** In a production environment logging in with a private key without passphrase would mean a security risk. It's not advisable to use this setup in a production environment. But to make this lab less complex, we decided to do it anyway. In a production ansible environment, you can encrypt your passwords and passphrases with ansible vault (shown in a later lab) or ansible tower (which is also one big vault).

### Task 3.1:  Playbook creation

In the playbook we will use the module ``authorized_key`` to install the SSH public key on the Client. The .ssh directory contains the private and public key created in Lab 0.

* Check if the SSH keys are installed (see lab 0, preparations):

  ``$ ls ~/.ssh/``

  ```
  id_rsa  id_rsa.pub
  ```

The file ``id_rsa.pub`` is the public key. The file ``id_rsa`` is the private key. 
 
* Create the playbook:

  ``$ vi workshop.yml``
  
* Add to your playbook (watch your usernumber!):

  ```
  ---
  - hosts: workshop

    tasks:
    - name: "Ensure authorized key is installed for user"
      authorized_key:
        user: userXX
        state: present
        key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
  ```

**Tip:** Playbooks work with Yaml files. Yaml works with indents of 2 spaces. If you ever worked with python, you will recognize this.

The playbook should be fairly readble. Even without Ansible knowledge, you should be able to guess what this playbook will do. The summary:
* The playbook will be executed on all clients in the group ``workshop``.
* Het playbook contains one single task.
* After ``name`` is a description of the task.
* Te module ``authorized_key`` is used to install  the ``key`` for ``user`` ``userXX``. The file ``~/.ssh/id_rsa.pub`` is used for this.

In the documentation you can find more details about the module ``authorized_key``. See https://docs.ansible.com/ansible/latest/modules/authorized_key_module.html.


### Task 3.2: Run the playbook

Om het playbook uit te voeren gebruiken we het commando: ``ansible-playbook``:

``$ ansible-playbook --ask-pass workshop.yml``


```
SSH password: 

PLAY [all] ********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************************
ok: [web1]
ok: [lb]
ok: [web2]

TASK [Ensure authorized key is installed for user {{ ANSIBLE_USER }}] *************************************************************************************************************************************************************************
changed: [lb]
changed: [web2]
changed: [web1]

PLAY RECAP ********************************************************************************************************************************************************************************************************************
lb                         : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

### Task 3.3: Running the playbook again

Because the authorized key is installed, you can login to the system without password.

* Check the connection without password (replace ``<hostname>`` with the hostname of your client):

``$ ssh -l {{ ANSIBLE_USER }} {{ ANSIBLE_CLIENT_1 }}``


```
Last login: Wed Jun 19 09:43:20 2019 from 13.94.128.79
[{{ ANSIBLE_USER }}@{{ ANSIBLE_CLIENT_1 }} ~]$ 
```


* Log out again with ``exit``:

``$ exit``


```
logout
Connection to {{ ANSIBLE_CLIENT_1 }} closed.
```

* Start the playbook, but without the parameter ``--ask-pass``:

``$ ansible-playbook workshop.yml``



```
PLAY [all] ********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************************
ok: [lb]
ok: [web2]
ok: [web1]

TASK [Ensure authorized key is installed for user user01] *********************************************************************************************************************************************************************
ok: [lb]
ok: [web2]
ok: [web1]

PLAY RECAP ********************************************************************************************************************************************************************************************************************
lb                         : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web1                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

The playbook will have no changes, as the public key is already installed. If you want to replace the public key later, all you have to do is generate a new one, and run the playbook. Ansible will notice the difference in the files and will change the file.

Next stp: [Lab 4 Playbook - Apache Installation ](04_NL_playbook_apache_installation.md)
