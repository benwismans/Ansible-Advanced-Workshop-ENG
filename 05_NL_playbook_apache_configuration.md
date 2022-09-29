## Lab 5: Playbook - Configuration Apache 

In this lab we will configure Apache with everything that's needed to actually show web pages.

### Task 5.1: Apache configure

To make sure Apache is started during boot, the Apache services needs to be ``enabled``. In the same task, we also make sure that the service is ``started``.

We will also configure firewalld to open up the ``http`` and ``https`` ports. We do this with the boolean ``immediate`` set, but also save the configuration with ``permanent``.

TIP: Also take a look at the documentation of the modules, so you know what all parameters do:  https://docs.ansible.com/ansible/latest/modules/systemd_module.html and https://docs.ansible.com/ansible/latest/modules/firewalld_module.html.

* Add to the playbook ``webservers.yml``:

```
  - name: "Ensure apache service is enabled and started"
    systemd:
      name: httpd
      enabled: true
      state: started
      
  - name: "Ensure firewalld service is enabled and started"
    systemd:
      name: firewalld
      enabled: true
      state: started

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

**NOTE:** In the ``firewall`` task we use a loop ``with_items``.  More info about loops: https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html

**TIP:** Ansibe works with a ``desired state``. If the Apache service hadn't been started yet, it will be started. As we put in our playbook that the ``state`` should be ``started``. Ansible is ``idempotent``. This means it will first check if the state is in the desired state already. If httpd had already been started, it would just skip the task. 


### Task 5.2: Webpage index.html

To actually show a webpage, we create an index.html in the default webserver root (``/var/www/html``). To show the difference between the 2 webservers in a a later state, we use the variable ``{{ ansible_fqdn }}``. 

* Add to the ``webservers.yml``:

```
  - name: "Ensure index.html is installed"
    copy:
      content: "This is server {{ ansible_fqdn }}"
      dest: "/var/www/html/index.html"
```

**TIP:** Ansible gathers ``facts``of each system. These ``facts`` contain useful values of a host. The module ``setup`` is responsible for these ``facts``. With an ``adhoc`` command you can also see the facts: ``ansible -m setup web1``. See https://docs.ansible.com/ansible/latest/modules/setup_module.html

* Start the playbook. The webservers are now ready!

``$ ansible-playbook webservers.yml``

### Task 5.3: Test

```
PLAY RECAP ********************************************************************************************************************************************************************************************************************
web1                       : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2                       : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

* Open in your webbrowser de URL van de eerste webserver: http://{{ <hostname2> }}.
* Check if the page is correct:

``This is server {{ <hostname2> }}``


* The second webserver as well: http://{{ <hostname3> }}:
``This is server {{ <hostname3> }}``

Next Step: [Lab 6 Role - HA Proxy](06_NL_role_haproxy.md)
