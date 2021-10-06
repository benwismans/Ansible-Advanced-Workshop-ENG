## Lab 9: Role - Templates

Voor configfiles is het gebruikelijk om een template te maken. Voor templates wordt ``jinja2`` gebruikt. Dit is een taal welke in Python gebruikt wordt voor templates. Zie http://jinja.pocoo.org/.

In de praktijk wordt vaak elke waarde in een configfile in een variable gezet. De default settings van de configfile worden dan in de ``defaults/main.yml`` gezet.

### Task 9.1: Default variablen definiëren

We gaan de configfile van de role uit link:08_NL_role_create[Lab 8] ombouwen naar een template.

* Ga naar de directory van de role ``chrony``:
+
[source,lang=bash]
----
$ cd ~/.ansible/roles/chrony
----
+
* Vul de file ``defaults/main.yml`` met:
+
[source,role=copypaste]
----
---
ntp_servers:
- 0.centos.pool.ntp.org
- 1.centos.pool.ntp.org
- 2.centos.pool.ntp.org
- 3.centos.pool.ntp.org

ntp_driftfile: /var/lib/chrony/drift
ntp_makestep: "1.0 3"
ntp_rtcsync: yes
ntp_logdir: /var/log/chrony
----

[NOTE] 
====
Merk op dat de variable ``ntp-servers`` een lijst is. Deze kan dus eenvoudig aangevuld worden met extra NTP servers. De variable ``ntp_rtcsync`` is een ``boolean``. We zullen later in de template dus met een ``if`` statement de waarde moeten controleren
====

### Task 9.2: Template maken

* Verwijder eerst de file ``files/chrony.conf``. Deze gaan we namelijk omzetten naar een template.
+
[source,lang=bash]
----
$ rm files/chrony.conf
----
+
* Maak een nieuwe file ``templates/chrony.conf.j2`` en vul deze met:
+
[source,role=copypaste]
----
# Record the rate at which the system clock gains/losses time.
driftfile {{ WORKAROUND_ROLE_VAR1 }}

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep {{ WORKAROUND_ROLE_VAR2 }}

# Specify directory for log files.
logdir {{ WORKAROUND_ROLE_VAR3}}
----
+
* Voor de NTP servers maken we een ``for`` loop. Voeg toe:
+
[source,role=copypaste]
----
{{ WORKAROUND_ROLE_VAR4 }}
{{ WORKAROUND_ROLE_VAR5 }}
{{ WORKAROUND_ROLE_VAR6 }}
----
+
* Voor ``rtcsync`` gebruiken we een ``if``. Voeg toe:
+
[source,role=copypaste]
----
{{ WORKAROUND_ROLE_VAR7 }}
rtcsync
{{ WORKAROUND_ROLE_VAR8 }}
----
+
* In de ``tasks/main.yml`` moet nu de ``copy`` module aangepast worden naar de ``template`` module. Dit is niet complex, omdat de ``template`` module erg lijkt op de ``copy`` module. Vervang simpelweg ``copy`` door ``template``.
* Pas daarnaast de filename in de ``src`` aan van ``chrony.conf`` naar ``chrony.conf.j2``
* Voer het playbook uit en controleer op een client of de template is toegepast op de config file ``/etc/chrony.conf``.
+
[source,lang=bash]
----
$ ssh {{ ANSIBLE_CLIENT_1 }}
$ cat /etc/chrony.conf
----
+
[source]
----
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
...
...
----


### Task 9.3: Role variablen in een playbook

* Zet de volgende variablen in het playbook:
+
[source,role=copypaste]
----
ntp_servers:
- 0.europe.pool.ntp.org
- 1.europe.pool.ntp.org
- 2.europe.pool.ntp.org
- 3.europe.pool.ntp.org

ntp_rtcsync: no
----
+
TIP: Kijk eventueel in ``Task 6.2`` in link:06_NL_role_haproxy[Lab 6] terug hoe je variablen voor een role in een playbook zet.
+
* Voer het playbook uit en controleer op een client of de wijzigingen goed zijn doorgevoerd.
+
[source,lang=bash]
----
$ ssh {{ ANSIBLE_CLIENT_1 }}
$ cat /etc/chrony.conf
----
+
[source]
----
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.europe.pool.ntp.org iburst
server 1.europe.pool.ntp.org iburst
server 2.europe.pool.ntp.org iburst
server 3.europe.pool.ntp.org iburst
...
...
----
+
TIP: Merk op dat de nameservers gewijzigd zijn van X.centos.pool.ntp.org naar X.europe.pool.ntp.org



