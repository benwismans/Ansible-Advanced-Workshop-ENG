## Lab 4: Playbook - Install Apache

In this lab we will install Apache on the webservers. Apache should NOT be installed on the loadbalancers. That's why we only choose the group ''webservers'' in the playbook.

### Task 4.1: Webservers playbook

* Create a new playboo ``webservers.yml``:

``$ vi webservers.yml``


* Add to playbook:

```
---
- hosts: webservers

  tasks:
  - name: "Ensure apache is installed"
    yum:
      name: httpd   
      state: latest
```

* Test your playbook:

``$ ansible-playbook webservers.yml``

CAUTION: The playbook fails.

* Look good at the error:

```
PLAY [webservers] *************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************************************
ok: [web2]
ok: [web1]

TASK [Ensure apache is installed] *********************************************************************************************************************************************************************************************
fatal: [web2]: FAILED! => {"changed": true, "changes": {"installed": ["httpd"], "updated": []}, "msg": "You need to be root to perform this command.\n", "rc": 1, "results": ["Loaded plugins: fastestmirror, langpacks\n"]}
fatal: [web1]: FAILED! => {"changed": true, "changes": {"installed": ["httpd"], "updated": []}, "msg": "You need to be root to perform this command.\n", "rc": 1, "results": ["Loaded plugins: fastestmirror, langpacks\n"]}

PLAY RECAP ********************************************************************************************************************************************************************************************************************
web1                       : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
web2                       : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   
```

If you pay attention to the errors, it seems like there is a permission issue. If you would normaally install the packages with ``yum`` you would use ``sudo`` for this. Within ansible there is no difference. We will instruct Ansible to use ``sudo``.

**TIP:**  Besides ``sudo``  Ansible also other methods of elevation. See: https://docs.ansible.com/ansible/latest/user_guide/become.html.

### Task 4.2: Start task with sudo permissions

In Ansible the variable ``become`` is responsible for running a playbook with elevated permissions. You use ``become_method`` for this.

* Adjust your playbook:

``$ vi webservers.yml``

* Add under ``hosts``:

```
  become: true
  become_method: sudo
```

* Start the playbook again and check if it works.

``$ ansible-playbook webservers.yml``


```
PLAY RECAP ********************************************************************************************************************************************************************************************************************
web1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

**TIP:** Te Ansible clients in this workshop were configured in a way that sudo will not ask for a password (``NOPASSWD: ALL`` in the sudoers file). That's why we can start the playboo with any ``privilege escalation password`` (``--ask-become-pass`` or ``-K``). In production environments it is common to use sudo with a password. 

Next step: [Lab 5 Playbook - Apache Configuration](05_NL_playbook_apache_configuration.md)
