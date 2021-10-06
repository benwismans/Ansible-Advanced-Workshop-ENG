## Lab 7: Ansible Vault

Ansible Vault is een functie van Ansible waarmee je gevoelige gegevens, zoals wachtwoorden of private keys (bijvoorbeeld voor SSL certificaten), kunt versleutelen. Dit maakt het mogelijk om deze gegevens toch in een ``SCM`` te zetten. Het is namelijk niet de bedoeling dat je deze gegevens in plain tekst in een ``SCM`` zet.

Vault variablen worden door Ansible automatisch ontsleutelt, mits het ``vault password`` bekend is. Een vault variable begint altijd met ``$ANSIBLE_VAULT;1.1;AES256``.


### Task 7.1: Bestand encrypten

TIP: ansible-vault gebruikt standaard de editor ``vi``. Mocht je liever ``nano`` gebruiken, dan kun je de editor aanpassen door de environment variable ``EDITOR`` te vullen met: ``/bin/nano``. Gebruik hiervoor het commando: ``export EDITOR=/bin/nano``.

* Maak een nieuw bestand:
+
[source,lang=bash]
----
$ ansible-vault create foo
----
+
* Bedenk zelf een wachtwoord:
+
[source]
----
  New Vault password:
  Confirm New Vault password:
----
+
* Zet een geheime boodschap in het bestand en sla deze op (in ``vi`` gaat dat met ``:wq``).
* Bekijk het bestand:
+
[source,lang=bash]
----
$ cat foo
$ANSIBLE_VAULT;1.1;AES256
36323433633666373164336366383438613964306431646537643863343762663465376265326337
6335663530313034653733323431376236316637643536610a323537336233373139363538383438
65633532333866356339333238653964633938363661353331373237343366306239313632623935
3738363065653864660a326666316531643837366161656531366239376338343534336230613832
6563
----


### Task 7.2: Encrypted bestand bewerken

* Edit het bestand:
+
[source,lang=bash]
----
$ ansible-vault edit foo
----
+
* Ansible vault vraagt om het wachtwoord. Alleen als het wachtwoord correct wordt ingevoerd zal de editor openen
+
[source]
----
Vault password:
----
+
* Maak een wijziging en sla het bestand weer op.

### Task 7.3: Encrypted bestand inzien

* Bekijk het bestand:
+
[source,lang=bash]
----
$ ansible-vault view foo
----
+
* Ansible vault vraagt om het wachtwoord. Alleen als het wachtwoord correct wordt ingevoerd zal het bestand weergegeven worden:
+
[source,lang=bash]
----
Vault password:
<inhoud van het bestand>
----

### Task 7.4: Wachtwoord wijzigen van een encrypted bestand

* Pas het wachtwoord aan:
+
[source,lang=bash]
----
$ ansible-vault rekey foo
Vault password:
New Vault password:
Confirm New Vault password:
Rekey successful
----

### Task 7.5: Encrypted bestand in je playbook gebruiken

Ansible herkent zelf of een bestand encrypt is en zal deze automatisch decrypten. Dit werkt voor alle modules. Voorwaarde is wel dat het ``vault password`` meegegeven wordt in het ``ansible-playbook`` commando. De parameter daarvoor is ``--ask-vault-pass``.

* Vul je playbook ``workshop.yml`` aan met:

+
[source,role=copypaste]
----
  - name: "Ensure foo is copied and decrypted"
    template:
      src: foo
      dest: /home/{{ ANSIBLE_USER }}/foo 
----
+
* Voer je playbook uit (Let er op dat je de parameter ``--ask-vault-pass`` mee geeft):
+
[source,lang=bash]
----
$ ansible-playbook workshop.yml --ask-vault-pass
----
+
* Log in op een client:
+
[source,lang=bash]
----
$ ssh {{ ANSIBLE_CLIENT_1 }}
----
+
* Controleer of het bestand decrypted op je Ansible client ({{ ANSIBLE_CLIENT_1 }}) staat:
+
[source,lang=bash]
----
$ cat foo
<inhoud van het bestand>
----  
+
TIP: Vergeet niet uit te loggen uit {{ ANSIBLE_CLIENT_1 }}
  
Volgende Stap: [Lab 8 Role - Rol aanmaken](08_NL_role_create.md)
