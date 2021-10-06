## Lab 6: Playbook - Role - HAProxy

Nu we 2 webservers hebben, is het tijd om een loadbalancer te installeren die er voor zorgt dat beide webservers gebruikt worden. We gaan daarvoor een ``role`` gebruiken. De ansible server waar je op werkt, wordt ook de loadbalancer.

Voor bijna elke uitdaging is wel een kant-en-klare rol te vinden op Ansible Galaxy (https://galaxy.ansible.com/). Het nadeel is echter dat er te veel roles te vinden zijn, waardoor het lastig is om de parels te vinden. Let bijvoorbeeld op het aantal downloads en het aantal sterren. 

**TIP:** Bekijk altijd de broncode van de role die je op Ansible Galaxy hebt gevonden en controleer of deze precies doet wat je zoekt. Indien nodig kun je de role altijd aanpassen.


Voor het installeren van HAproxy kun je bijvoorbeeld zoeken op ``haproxy``. De role ``geerlingguy.haproxy`` (https://galaxy.ansible.com/geerlingguy/haproxy)  lijkt precies te doen wat we willen. Deze gaan we installeren via Ansible Galaxy. 

**TIP:**
Naast het kijken naar het aantal stars of downloads kun je ook kijken of een ontwikkelaar meerdere roles heeft gebouwd. Enkele bekende ontwikkelaars zijn:

* https://galaxy.ansible.com/debops
* https://galaxy.ansible.com/geerlingguy
* https://galaxy.ansible.com/Oefenweb


### Task 6.1: Role installeren

We gaan een Ansible Role gebruiken om een user account aan te maken.  Doordat de tasks in de role al zijn geschreven hoef je alleen maar de role te installeren en met parameters de role aan te sturen.

* Installeer de role via Ansible Galaxy:

``$ ansible-galaxy install geerlingguy.haproxy``

De role wordt geïnstalleerd naar de ``.ansible/roles`` directory in je home directory.

* Controleer of de role is geinstalleerd in ``.ansible/roles``:

```
$ ls ~/.ansible/roles
geerlingguy.haproxy
```

* Bekijk de role:

```
$ cd ~/.ansible/roles/geerlingguy.haproxy
$ ls
LICENSE  README.md  defaults  handlers  meta  tasks  templates  tests
```

Een beschrijving van de onderdelen van een role vind je terug in de documentatie van Ansible op: https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html.

**TIP:** Het vervolg van de workshop wordt vanuit je home directory uitgevoerd. Ga terug naar je home directory met het commando: ``$ cd``.

### Task 6.2: Playbook maken met role

* Maak een nieuw playbook ``loadbalancer.yml``:

```
---
- hosts: loadbalancer
  become: true
  become_method: sudo

  vars:
    haproxy_backend_servers:
    - name: web1
      address: {{ <hostname2> }}:80
    - name: web2
      address: {{ <hostname3> }}:80

  roles:
  - geerlingguy.haproxy
```

* Voer het playbook uit:

``$ ansible-playbook loadbalancer.yml``

IMPORTANT: De role mist de firewall configuratie. We zetten http tijdelijk open in de firewall met een adhoc commando. Later zullen we de role aanpassen en de firewall configuratie in de role meenemen.

* Zet http open in de firewall:

``$ ansible -b -m firewalld -a "service=http state=enabled" loadbalancer``

**NOTE:** Zoek met ``ansible --help`` uit wat de parameters ``-b``, ``-m`` en ``-a`` betekenen


### Task 6.3: Werking testen

* Open de URL: http://{{ <hostname1> }}/

NOTE: Je zou verwachten dat je de ene keer de pagina van ``web1`` krijgt en de andere keer van ``web2`` als je de pagina ververst. In HAProxy is door de role echter een cookie geconfigueerd die er voor zorgt dat je telkens op dezelfde server uit komt. Je zult dus met meerdere browsers moeten testen, of de ``incognito`` modus van je browser moeten gebruiken om de werking goed te testen. Of via CLI met curl.

Je kunt een adhoc commando gebruiken om httpd te stoppen / starten op de webservers:

``$ ansible -b -m systemd -a "name=httpd state=stopped" web1``

**TIP:** Gebruik het bovenstaande adhoc commando om httpd op ``web1`` of ``web2`` te stoppen of te starten. Controleer de werking via de URL: http://{{ ANSIBLE_CLIENT_3 }}/

* Zorg dat beide webservers weer gestart zijn.

### Task 6.4: Role aanpassen

We willen graag de cookie functionaliteit uit de configuratie verwijderen, omdat we later een non-disruptive update gaan uitvoeren met Ansible (zie Lab 8). Daarnaast mist de role de firewall configuratie.

* Bewerk de template ``haproxy.cfg.j2`` om de cookie settings te verwijderen:

```
$ cd ~/.ansible/roles/geerlingguy.haproxy/
$ vi templates/haproxy.cfg.j2 
```

* Verwijder de regel (in het onderdeel ``backend``):

```
    cookie SERVERID insert indirect
```

* Verwijder ``cookie`` en de laatste {{ WORKAROUND_BACKENDNAME }} uit de server regel (een-na-laatste regel):

```
    {{ WORKAROUND_BACKENDSERVER }}
```

* Voeg de firewall configuratie toe aan de task:

``$ vi tasks/main.yml``


```
- name: "Ensure firewall is configured"
  firewalld:
    service: "{{ WORKAROUND_ITEM }}"
    permanent: yes
    immediate: yes
    state: enabled
  with_items:
  - http
  - https
```

* Voer het playbook uit

``ansible-playbook loadbalancer.yml``

* Open de URL: http://{{ ANSIBLE_CLIENT_3 }}/ en test of je nu wel van beide servers een antwoord krijgt.

Volgende Stap: [Lab 7 Vault - Encryptie gebruiken](07_NL_vault.md)
