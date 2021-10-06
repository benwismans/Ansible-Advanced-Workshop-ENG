## Introductie

Welkom bij de Ansible Advanced Workshop. In deze workshop gaan we een loadbalancer met 2 webservers inrichten. Vervolgens gaan we een Ansible playbook maken om deze omgeving automatisch, zonder onderbreking, te updaten.

### Benodigdheden

Voor deze workshop heb je nodig:

* Laptop met een SSH client (bijvoorbeeld putty: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
* Een Linux server met SSH toegang en Ansible Server geinstalleerd. 
* Twee basis Linux servers.

### Uitvoering

* Alle acties worden uitgevoerd vanuit de home directory van de Ansible server (tenzij anders aangegeven).
* Instructies voor commando's worden vooraf gegaan met een prompt teken ($) en staan in een tekst blok. Dit teken is onderdeel van de opdracht prompt en hoort niet bij het commando (wie kent de dos prompt ``C:\>`` nog?). Bijvoorbeeld (Het commando in het voorbeeld is dus ``ls``):


  ``$ ls``
  
- De output wordt ook in een tekstblok weergegeven. Bijvoorbeeld:
  ```
  user@vm01:~ $ ls -l
  total 3
  -rw-r--r-- 1 pi pi   84 Feb 14 13:30 ansible.cfg
  -rw-r--r-- 1 pi pi  979 Feb 14 17:07 brocade.yml
  -rw-r--r-- 1 pi pi  884 Feb 14 22:07 cisco.yml
  ```

Volgende stap: [Lab 2 Inventory file aanmaken](02_NL_inventory.md)

