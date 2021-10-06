## Extra: Windows Part 2


### Task 12.1: IIS Webserver role installeren

De Ansible module ``win_feature`` kan gebruikt worden om Roles of Features toe te voegen aan Windows. In deze task gaan we de IIS Webserver installeren en configureren.

* Vul het playbook ``windows.yml`` aan met:
+
[source,role=copypaste]
----
  - name: Install IIS Web-Server with sub features and management tools
    win_feature:
      name:
        - Web-Server
      include_sub_features: yes
      include_management_tools: yes
----
+
TIP: Je kunt met het PowerShell commando: ``Get-WindowsFeature`` een lijst opvragen met de namen alle roles en features.
+
* Voer het playbook uit
+
NOTE: De installatie van de role duurt even. Ansible zal pas verder gaan met met de volgende task als de feature is geïnstalleerd.
+
* Gebruik de ``win_copy`` module om een index.html te schrijven naar ``C:\inetpub\wwwroot\index.html`` met de inhoud: ``This is server {{ WORKAROUND_FQDN }}``.
+
TIP: Kijk terug in link:05_NL_playbook_apache_configuration[Lab 5 Playbook - Apache configureren] hoe je een index.html aan kunt maken.
+
NOTE: Let er op dat je single quotes ``'`` gebruikt voor Windows paden (of ``\\`` met double quotes ``"``).
+
* Test de werking via de url: http://{{ ANSIBLE_CLIENT_4 }}


### Task 12.2: Packages installeren met Chocolatey

Bijna elke Linux distributie heeft wel een package manager. Voor Debian varianten is dat ``apt`` en op Red Hat varianten is dat ``yum``. Voor Windows is er standaard geen package manager. Maar er bestaat wel 3rd party software die deze rol kan vervullen. Een voorbeeld daarvan is ``Chocolatey`` (zie: https://chocolatey.org/). Ansible heeft een module om packages via Chocolatey te installeren. Een overzicht van deze packages is te vinden op: https://chocolatey.org/packages. Ideaal voor installatie van standaard software die op bijna elk Windows systeem is te vinden (bijvoorbeeld Adobe Acrobat Reader: ``adobereader`` of Java: ``jre8``).

* Vul je playbook ``windows.yml`` aan met:
+
[source,role=copypaste]
----
  - name: Install packages with Chocolatey
    win_chocolatey:
      name: '{{ WORKAROUND_ITEM }}'
    with_items:
      - adobereader
      - putty
      - windirstat
      - googlechrome
----
+
* Voer het playbook uit en controleer of de software is geïnstalleerd.
+
TIP: Chocolaty is standaard niet geïnstalleerd. De module ``win_chocolatey`` installeerd automatisch Chocolatey. Omdat dit nog niet geïnstalleerd was, krijg je de warning ``Chocolatey was missing from this system, so it was installed during this task run.``

### Task 12.3: Windows Update

Met Ansible kun je Windows updates geautomatiseerd uitvoeren. In deze workshop gaat het maar om 1 server, maar dit kan natuurlijk op meerdere servers tegelijk. Zelfs automatisch herstarten is mogelijk.

* Vul je playbook ``windows.yml`` aan met:
+
[source,role=copypaste]
----
  - name: Install Windows Update
    win_updates:
      category_names:
        - SecurityUpdates
      reboot: yes
----
+
* Voer het playbook uit


