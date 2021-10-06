## Lab 8: Role - Zelf bouwen

In Ansible is het een best-pratice om te werken met roles. Het is daarom goed om te weten hoe je zo'n role zelf kunt bouwen. In dit lab gaan we een role bouwen om ``chrony`` te configureren. 

NOTE: In RHEL 7 / CentOS 7 is chrony de standaard daemon voor het network time protocol (NTP). Er zijn op internet genoeg artikelen te vinden die het verschil tussen chrone en ntpd uitleggen.

Een role bestaat uit de volgende onderdelen (directories):

* ``tasks``: Alle taken voor de role.
* ``handlers``: Bevat de handlers die met ``notify`` uitgevoerd kunnen worden. Wordt veel gebruikt om services te herstarten. Zie https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#task-and-handler-organization-for-a-role voor meer details.
* ``defaults``: Default variablen voor de role. Omdat default variablen de laagste prioriteit hebben, zijn deze makkelijk te overrulen.
* ``vars``: Variablen voor de role. Variablen in ``vars`` hebben een hogere prioriteit dan variablen in ``defaults``.
* ``files``: Bevat files die gebruikt worden door de role. Deze kunnen bijvoorbeeld in de copy module gebruikt worden.
* ``templates``: Bevat files met variablen. Voor templates wordt Jinja2 als templating taal gebruikt.
* ``meta``: Bevat metadata voor de role, zoals bijvoorbeeld: Auteur, ondersteunde platformen of afhankelijkheden met andere roles.

De directories ``tasks``, ``handlers``, ``defaults``, ``vars`` en ``meta`` dienen een file ``main.yml`` te bevatten. In deze file wordt het betreffende onderdeel beschreven. Elke ``main.yml`` begint met ``---``. Ook als de file verder geen inhoud heeft.

```
---
```

### Task 8.1: Lege rol aanmaken

Met het commando ``ansible-galaxy init --offline <rolename>`` maak je een skelet aan voor de nieuwe role.

NOTE: Roles worden default in ``/etc/ansible/roles`` of ``~/.ansible/roles/`` (in je home directory) gezet.

* Maak een nieuwe role aan:

```
$ cd ~/.ansible/roles/
$ ansible-galaxy init --offline chrony
```

* Met ``find`` krijg je een mooi overzicht van alle directories en files in het skelet:
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

### Task 8.2: Handler file maken

In de file ``handlers/main.yml` wordt de handler gedefinieerd. Een ``handler`` wordt alleen uitgevoerd als een task een change uitvoert. Daarom wordt een ``handler`` vaak gebruikt om services te herstarten.

* Vul de file ``handlers/main.yml`` met:
```
---
- name: Restart chrony
  systemd:
    name: chronyd
    state: restarted
```
**TIP:** Deze handler herstart ``chronyd`` wanneer de task, die de handler aanroept, een wijziging oplevert.

### Task 8.3: Default variables definiëren

De default variablen worden in de file ``defaults/main.yml`` gezet. Wanneer je een bepaalde service installeert en configureert, is het gebruikelijk om de naam van de service in een default variable te zetten. Daardoor kun je de role later nog makkelijk hergebruiken. Je hoeft immers alleen de variable in ``defaults/main.yml`` aan te passen. De task hoef je dan nauwelijks te bewerken.

* Zet de volgende variablen in de file ``defaults/main.yml``:
```
---
chrony_package: chrony
chrony_service: chronyd
```

### Task 8.4: Config file maken

* Maak een nieuwe file ``files/chrony.conf`` en vul deze met:
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

De laatste stap is het aanmaken van de tasks. Dit werkt vergelijkbaar met de ``tasks`` van een ``playbook``.

* Bewerk de file ``tasks/main.yml``:
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

**TIP:** De regel ``notify: Restart chrony`` zorgt er voor dat de handler uitgevoerd wordt als de config file gewijzigd is.

### Task 8.6: Playbook

* Maak een nieuw playbook ``chrony.yml`` om de role uit te voeren. Kijk terug in link:06_NL_role_haproxy[Lab 6 Role - HAProxy] hoe je een playbook met een role maakt.

**[NOTE]**
Je was bezig in de directory ``~/.ansible/roles``. Let er op dat je eerst weer naar je home directory gaat voordat je het playbook aanmaakt.
```
$ cd ~
```
* Voer het playbook uit

**NOTE:** Het playbook zal een change opleveren op de ``task`` ``Ensure chrony is configured``. Daardoor zal ook de handler ``Restart chrony`` uitgevoerd worden. De handler wordt altijd aan het eind van het playbook uitgevoerd.


Volgende Stap: [Lab 9 Role - Templates gebruiken](09_NL_templates.md)
