## Lab 3: Playbook - User configureren

Een playbook is een beschrijving hoe een systeem ingericht zou moeten zijn. Dit playbook bestaat uit een lijst met stappen. Elke stap controleert de huidige toestand en past deze, indien nodig, aan. Komt het systeem al overeen met de beschrijving van de stap, dan doet Ansible niets.

TIP: Probeer het playbook zo in te richten dat er geen changes meer worden gemaakt, wanneer het playbook voor de 2e maal uitgevoerd wordt. Daarnaast is het gebruikelijk om een playbook zodanig te maken dat deze meerdere malen uitgevoerd kan worden, zonder dat dit problemen geeft.

In dit lab maken we een playbook voor het installeren van de public key voor SSH. Na het installeren van deze public key kun je met de private key (dus zonder wachtwoord) inloggen op de Ansible clients.

NOTE: In een productie omgeving levert het een beveiligings risico op, wanneer er geen Passphrase is geconfigueerd op de Private key. Het is daarom niet aan te raden om Private keys zonder een Passprhase te gebruiken in een productie omgeving. Om dit lab minder complex te maken, is de Passphrase voor de Private key leeg gelaten.

### Task 3.1:  Playbook aanmaken

In het playbook gaan we de module ``authorized_key`` gebruiken om de SSH public key op de Ansible clients te installeren. In de verborgen directory .ssh is de public en de private key voor geïnstalleerd.

* Maak het playbook aan:

[source]
----
$ vi workshop.yml
----

* Vul het playbook met:

[source,role=copypaste]
----
---
- hosts: all

  tasks:
  - name: "Ensure authorized key is installed for user {{ ANSIBLE_USER }}"
    authorized_key:
      user: {{ ANSIBLE_USER }}
      state: present
      key: "{{ WORKAROUND_PUBKEY }}"
----


TIP: Playbooks werken met Yaml files. Voor de werking van Yaml files is het belangrijk dat het inspringen van de regels nauwkeurig gebeurd. Het is gebruikelijk om dit met 2 (of 4) spaties te doen. Als je ooit met Python hebt gewerkt, dan zul je dit herkennen. 

Als het goed is, valt op dat het playbook redelijk leesbaar is. Zelfs zonder kennis van Ansible is redelijk in te schatten wat dit playbook uit zal voeren. De samenvatting:

* Het playbook zal uitgevoerd worden op alle clients.
* Het playbook bestaat uit een enkele taak.
* Met ``name`` wordt beschreven wat deze taak doet.
* De module ``authorized_key`` wordt gebruikt om voor de ``user`` ``{{ ANSIBLE_USER }}`` de ``key`` te installeren. Daarbij wordt de file ``.ssh/id_rsa.pub`` gebruikt.

TIP: In de documentatie vind je meer details over de module ``authorized_key``. Zie https://docs.ansible.com/ansible/latest/modules/authorized_key_module.html.

### Task 3.2: Het playbook starten

Om het playbook uit te voeren gebruiken we het commando: ``ansible-playbook``:

[source]
----
$ ansible-playbook --ask-pass workshop.yml
----

[source]
----
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
----

### Task 3.3: Het playbook nogmaals starten

Omdat nu de Authorized key voor SSH op de Ansible clients is geïnstalleerd, kun je zonder wachtwoord inloggen op een Ansible client.

* Controleer of je zonder wachtwoord in kunt loggen:

[source]
----
$ ssh -l {{ ANSIBLE_USER }} {{ ANSIBLE_CLIENT_1 }}
----

[source]
----
Last login: Wed Jun 19 09:43:20 2019 from 13.94.128.79
[{{ ANSIBLE_USER }}@{{ ANSIBLE_CLIENT_1 }} ~]$ 
----


* Log direct weer uit met ``exit``:

[source]
----
$ exit
----

[source]
----
logout
Connection to {{ ANSIBLE_CLIENT_1 }} closed.
----

* Start het playbook, maar zonder de parameter ``--ask-pass``:

[source]
----
$ ansible-playbook workshop.yml
----


[source]
----
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
----

Het playbook zal nu geen changes opleveren. De public key is immers al geïnstalleerd. Mocht je later de public key willen vervangen, kun je simpelweg een nieuwe genereren en deze via Ansible opnieuw deployen. Ansible herkent dat het bestand is gewijzigd en zal daarvoor een change genereren.

