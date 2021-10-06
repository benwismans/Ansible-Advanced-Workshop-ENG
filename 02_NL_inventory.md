## Lab 2: Inventory file aanmaken

Een belangrijk onderdeel voor Ansible is de inventory file. In deze file wordt beschreven hoe de omgeving er uit ziet.

Alle acties worden uitgevoerd in de home directory van de **Ansible server**.

### Task 2.1: Inventory file aanmaken

In de inventory file wordt beschreven hoe Ansible je Ansible clients kan bereiken. Een Ansible inventory werkt altijd met een groep, welke tussen blokhaken wordt gezet: [ en ]. Onder de groep worden alle hosts omschreven. In dit geval gaat het om maar 1 host. Omdat het aanspreken van een host makkelijker gaat met een naam, dan met een IP adres, geven we de client een naam. Met de variable ansible_host koppelen we deze naam aan het juiste IP adres.

* Edit de file ``inventory``:

[source]
----
$ vi inventory
----

* Vul de inventory file met:

[source,role=copypaste]
----
[loadbalancer]
lb ansible_host=localhost
[webservers]
web1 ansible_host=<hostname2>
web2 ansible_host=<hostname3>
----

### Task 2.2: Ansible vertellen waar de inventory file staat
Ansible zoekt standaard in de volgende paden naar de inventory file:

* /etc/ansible/hosts
  
In het configuratie bestand ansible.cfg kan een alternatief pad geconfigueerd worden naar de inventory file. Ansible zoekt in de volgende paden naar de ansible.cfg:

* ansible.cfg (in de huidige directory)
* .ansible.cfg (in de home directory
* /etc/ansible/ansible.cfg

Door een ansible.cfg in dezelfde directory te zetten als het playbook (welke we in een later lab aanmaken), worden alle default instellingen overruled door de instellingen in deze ansible.cfg. We laten de inventory wijzen naar ``~/inventory`` (de ~ is een alias voor je home directory; de plek waar de inventory file is aangemaakt). Nu we toch bezig zijn, configureren we alvast de user waarmee we straks via Ansible inloggen op de clients. Deze user is hetzelfde als op de bastion: {{ ANSIBLE_USER }}. Verder schakelen we host_key_checking uit. 

* Maak een ansible.cfg aan:

[source]
----
$ vi ansible.cfg
----

* Vul de ansible.cfg met:

[source,role=copypaste]
----
[defaults]
inventory = ~/inventory
remote_user = {{ ANSIBLE_USER }}
host_key_checking = False
----

### Task 2.3: Test de werking
Ansible werkt met modules. Voor bijna elke functie is wel een module te vinden. Voor het aanmaken van een gebruiker wordt bijvoorbeeld de module ``user`` gebruikt. In onze eerste stap met Ansible gaan we de module ``ping`` gebruiken. Met de module ``ping`` kun je de verbinding met je clients testen. Anders dan je gewend bent van de ping commando's van je Operating System (bijvoorbeeld Windows of Linux), test de ``ping`` module niet alleen of de client bereikbaar is (icmp reply), maar controleert of Ansible daadwerkelijk in kan loggen op de client (voor Linux clients logt Ansible in met SSH). Zie https://docs.ansible.com/ansible/latest/modules/ping_module.html#ping-module.

[TIP]
====
Een overzicht van alle modules is terug te vinden in de online documentatie van Ansible op: https://docs.ansible.com/ansible/latest/modules/list_of_all_modules.html.
====

* Controleer of de inventory file en de ansible.cfg in je home directory staan:

[source]
----
$ ls
----

[source]
----
ansible.cfg  inventory
----
  
* Controleer of de Ansible module ``ping`` antwoord geeft:

[source]
----
$ ansible --ask-pass -m ping webservers
----

[source]
----
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
----

NOTE: In ons voorbeeld vraagt Ansible om een SSH password. In een geautomatiseerd scenario is het gebruikelijk om met SSH Autorized Keys te werken. In een later lab richten we de clients in met Autorized keys, zodat Ansible direct, zonder wachtwoord, in kan loggen op de clients. Omdat we nog geen Authorized keys hebben ingericht, geven we met ``--ask-pass`` de instructie om een SSH password te vragen.

In de inventory file hebben we de groep [webservers] gedefineerd. De ``ping`` module zal daarom alle hosts controleren. In ons lab hebben we 2 hosts gedefineerd: ``{{ ANSIBLE_CLIENT_1 }}`` en ``{{ ANSIBLE_CLIENT_2 }}``. Beide hosts zullen dus antwoorden. 

TIP: Het is ook mogelijk om een enkele host te testen. Met ``ansible --ask-pass -m ping web1`` wordt de module alleen maar op de host ``{{ ANSIBLE_CLIENT_1 }}`` uitgevoerd. Let er op dat je de alias gebruikt, in plaats van de hostname.

Volgende stap: [Lab 3 Playbook - User aanmaken](03_NL_playbook_user.md)
