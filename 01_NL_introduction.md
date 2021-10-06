## Introductie

Welkom bij de workshop Ansible-Engine. In deze workshop gaan we een loadbalancer met 2 webservers inrichten. Vervolgens gaan we een Ansible playbook maken om deze omgeving automatisch, zonder onderbreking, te updaten.

### Benodigdheden

Voor deze workshop heb je nodig:

* Laptop met een SSH client (bijvoorbeeld putty: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
* Een Linux server met SSH toegang en Ansible (in deze workshop de bastion server: ``{{ ANSIBLE_BASTION }}``)
* Sheet met account gegevens
** Gebruikersnaam: ``{{ ANSIBLE_USER }}``
** Wachtwoord: ``{{ ANSIBLE_PASSWORD }}``

### Uitvoering

* Alle acties worden uitgevoerd vanuit de home directory van de SSH server (tenzij anders aangegeven).
* Instructies voor commando's worden vooraf gegaan met een prompt teken ($) en staan in een tekst blok. Dit teken is onderdeel van de opdracht prompt en hoort niet bij het commando (wie kent de dos prompt ``C:\>`` nog?). Bijvoorbeeld (Het commando in het voorbeeld is dus ``ls``):

[source,bash]
----
$ ls
----
  
* De output wordt ook in een tekstblok weergegeven. Bijvoorbeeld:

[source,bash]
----
user01@bastion:~ $ ls -l
total 3
-rw-r--r-- 1 pi pi   84 Feb 14 13:30 ansible.cfg
-rw-r--r-- 1 pi pi  979 Feb 14 17:07 workshop.yml
----

NOTE: Bovenstaande is slechts een voorbeeld. De files zijn nog niet aanwezig op de SSH server.

Volgende stap: [Lab 2 - Inventory file aanmaken](02_NL_inventory.adoc)