## Lab 4: Playbook - Apache installeren

In dit lab gaan we Apache installeren op de webservers. Apache moet natuurlijk niet op de loadbalancer geïnstalleerd worden. Daarom selecteren we alleen de groep ''webservers'' in het playbook.

### Task 4.1: Webservers playbook

* Maak een nieuw playbook ``webservers.yml``:

[source]
----
$ vi webservers.yml
----

* Vul het playbook met:

[source,role=copypaste]
----
---
- hosts: webservers

  tasks:
  - name: "Ensure apache is installed"
    yum:
      name: httpd   
      state: latest
----

* Test je playbook:

[source]
----
$ ansible-playbook webservers.yml
----

CAUTION: Het playbook faalt

* Kijk goed naar de foutmelding:

[source]
----
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
----

Als je goed kijkt naar de foutmeldingen, dan lijkt het er op dat er een rechten probleem is. Wanneer je de packages met ``yum`` zou installeren, zou je daar ``sudo`` voor gebruiken. Met Ansible is dat eigenlijk niet anders. Om de packages te installeren, zullen we Ansible moeten instrueren om ``sudo`` te gebruiken.

TIP:  Naast ``sudo`` ondersteund Ansible ook andere methoden om meer rechten te verkrijgen. Zie https://docs.ansible.com/ansible/latest/user_guide/become.html.

### Task 4.2: Task met sudo rechten starten

In Ansible is de variable ``become`` verantwoordelijk voor het starten van een playbook met meer rechten. De methode daarvoor stel je in met ``become_method``.

* Bewerk je playbook:

[source]
----
$ vi webservers.yml
----

* Zet onder ``hosts``:

[source,role=copypaste]
----
  become: true
  become_method: sudo
----

* Start het playbook opnieuw en controleer of er nu wel voldoende rechten zijn om packages te installeren.

[source]
----
$ ansible-playbook webservers.yml
----

[source]
----
PLAY RECAP ********************************************************************************************************************************************************************************************************************
web1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
----

TIP: De Ansible clients zijn in deze workshop zo geconfigureerd dat sudo niet om een wachtwoord vraagt (``NOPASSWD: ALL`` in de sudoers file). Daarom kunnen we het playbook starten zonder ``privilege escalation password`` (``--ask-become-pass`` of ``-K``). In productie omgevingen is het echter gebruikelijk om sudo met een wachtwoord te starten. Met ``--ask-become-pass`` (of ``-K``) kun je dit wachtwoord aan Ansible doorgeven.

Volgende stap: [Lab 5 Playbook - Configuratie Apache](05_NL_playbook_apache_configuration.md)
