## Lab 6: Playbook - Role - HAProxy

Now that we have 2 webservers, it is time to install a loadbalancer that will use both these webservers. We will use a ``role`` for this. The ansible server you are working on, will also be the loadbalancer.

For almost every challenge there is a role already created on Ansible Galaxy. The disadvantage of this is that there are too many, which makes it difficult to find the good stuff. You can check the number of downloads and the ranking (in stars). 

**TIP:** You should always check the source code of a role you found on Ansible Galaxy to make sure it does what you want it to do. You can always adjust things if needed. 

For installing HAproxy you could search for ``haproxy``. The role ``geerlingguy.haproxy`` (https://galaxy.ansible.com/geerlingguy/haproxy)  seems to do whwat we want, so we will use that one.

**Tip** Next to looking at the stars of downloads you can also check if a developer built multiple roles. The ontic roles might not have a lot of stars, but the developer made dozens of roles. Some well known contributors are:
* https://galaxy.ansible.com/debops
* https://galaxy.ansible.com/geerlingguy
* https://galaxy.ansible.com/Oefenweb

### Task 6.1: Role installatiojn

We are going to use an Ansible Role to create the haproxy setup. Because the tasks are already in the role, we only need to install the role and adjust the parameters.

* Install the role via Ansible Galaxy:

``$ ansible-galaxy install geerlingguy.haproxy``

The role will be installed in ``.ansible/roles`` directory in your home directory.

* Check if the role is installed ``.ansible/roles``:

```
$ ls ~/.ansible/roles
geerlingguy.haproxy
```

* Check the role:

```
$ cd ~/.ansible/roles/geerlingguy.haproxy
$ ls
LICENSE  README.md  defaults  handlers  meta  tasks  templates  tests
```

A description of the ports of a role can be found in the documentation of Ansible on: https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html.

**Tip:** The remaining part of this workshop will be executed from the homedirectory of the user. So go back with the command ``$ cd``

### Task 6.2: Playbook with the role

* Create a new playbook ``loadbalancer.yml``:

```
---
- hosts: loadbalancer
  become: true
  become_method: sudo

  vars:
    haproxy_backend_servers:
    - name: web1
      address: {<hostname2>:80
    - name: web2
      address: <hostname3>:80

  roles:
  - geerlingguy.haproxy
```

* Run the playbook:

``$ ansible-playbook loadbalancer.yml``

IMPORTANT: The role misses the firewall configuration. We will quickly open the http service with an adhoc command. 

* Zet http open in de firewall:

``$ ansible -b -m firewalld -a "service=http state=enabled" loadbalancer``


### Task 6.3:  Testing

* Open de URL: http://<hostname1>/

**NOTE**: You would expect to get a response from ``web1`` and ``web2`` if you refresh the page. HAProxy has a coookie configured so you end up on the same server each time. You can test with different browsers or in ``incognito mode`` or with a ``curl`` command.

You could also use an adhoc command to stop the webserver on web1.
``$ ansible -b -m systemd -a "name=httpd state=stopped" web1``

* Make sure both webservers are up and running again!

### Task 6.4: Role adjustments
  
We would like to delete the cookie functionality out of HAProxy, because we will do a rolling ugprade leter on with Ansible (Lab 8).

* Adjust the template ``haproxy.cfg.j2`` to delete the cookie settings:

```
$ cd ~/.ansible/roles/geerlingguy.haproxy/
$ vi templates/haproxy.cfg.j2 
```

* Delete the line (in the part ``backend``):

```
    cookie SERVERID insert indirect
```

* Delete ``cookie`` and the laatste {{ backend.name }} in the line  (not the complete line!) What should remain is this:

```
    server {{ backend.name }} {{ backend.address }} check
```

* Add the firewall configuration in the task:

``$ vi tasks/main.yml``


```
- name: "Ensure firewall is configured"
  firewalld:
    service: "{{ item }}"
    permanent: yes
    immediate: yes
    state: enabled
  with_items:
  - http
  - https
```

* Run the playbook uit

``ansible-playbook loadbalancer.yml``

* Open the URL: http://hostname1/ and test.

Next Step: [Lab 7 Vault - Encryptie gebruiken](07_NL_vault.md)
