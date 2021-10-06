## Extra - Windows Part 1

Ansible is niet alleen voor Linux. Je kunt er inmiddels ook prima Windows mee configureren. Ansible heeft veel kant-en-klare modules om Windows te configureren en beheren. Denk bijvoorbeeld aan het installeren van extra roles of features, installatie van MSI pakketten of aanpassen van de firewall. Natuurlijk zijn er ook modules voor basis taken, zoals files bewerken / kopieëren. Mocht er toch geen standaard module zijn voor je probleem, dan kun je altijd terugvallen op de modules ``win_command`` of ``win_psexec`` (voor Powershell commando's). Een overzicht van alle modules vind je terug op: https://docs.ansible.com/ansible/latest/modules/list_of_windows_modules.html.

De Windows VM in deze workshop is al voorbereid op Ansible. De handleiding voor het voorbereiden van Windows op Ansible vind je terug op: https://www.ansible.com/blog/connecting-to-a-windows-host.

De Windows VM is via RDP bereikbaar. Gebruik daarvoor de volgende gegevens:

* Hostnaam: ``{{ ANSIBLE_CLIENT_4 }}``
* Gebruikersnaam: ``{{ ANSIBLE_USER }}``
* Wachtwoord: ``{{ ANSIBLE_PASSWORD }}``

### Task 11.1: Inventory aanpassen

De Windows VM moet natuurlijk toegevoegd worden aan de inventory file. Voor Windows zijn er enkele extra parameters nodig om er voor te zorgen dat Ansible ``winrm`` gebruikt in plaats van ``SSH``.

* Voeg toe aan je ``inventory`` file:
+
[source,role=copypaste]
----
[windows]
win ansible_host={{ ANSIBLE_CLIENT_4 }}

[windows:vars]
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
----
+
TIP: Voor Windows dient de ``ansible_connection`` aangepast te worden naar winrm (Windows Remote Management). De WinRM module gebruikt SSL om de verbinding te beveiligen. Het SSL certificaat dient dan wel vertrouwd te worden. In een enterprise omgeving is meestal een PKI infrastructuur aanwezig, waardoor alleen het root certificaat geïnstalleerd hoeft te worden. Alle certificaten, ondertekend met dit root certificaat, worden daarmee automatisch vertrouwd. In deze workshop hebben we geen PKI. We schakelen daarom de verificatie uit met de variable ``ansible_winrm_server_cert_validation``.
+
* Test de werking:
+
[source,lang=bash]
----
$ ansible -m win_ping --ask-pass windows
----
+
[source]
----
windows | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
----
+
TIP: Waar je voor Linux de module ``ping`` zou gebruiken, moet je voor Windows de module ``win_ping`` gebruiken. De werking is verder hetzelfde.



### Task 11.2: Files kopieëren

Voor de meeste basis modules is er ook een Windows alternatief. Enkele voorbeelden:

* win_copy - Bestanden kopieëren naar een remote locatie
* win_user - Lokale users aanmaken of bewerken
* win_regedit - Regedit entries bewerken
* win_file - Files of directories aanmaken of verwijderen
* win_lineinfile - Configuratie bestanden bewerken

Om bestanden te kopieëren gebruik je de ``win_copy`` module.

* Maak een testfile ``foo.txt`` en zet er een test string in
* Maak een nieuw playbook: ``windows.yml`` en voeg een task toe om de file te kopieeren naar de VM
+
[source,role=copypaste]
----
---
- hosts: windows

  tasks:

  - name: Install host file
    win_copy:
      src: foo.txt
      dest: 'C:\workshop\'
      backup: yes
----
+
TIP: De directory ``C:\workshop`` bestaat waarschijnlijk nog niet. In de handleiding van de ``win_copy`` module (https://docs.ansible.com/ansible/latest/modules/win_copy_module.html#win-copy-module) is terug te lezen dat ``win_copy`` deze directory automatisch aan zal maken als deze nog niet bestaat.
+
NOTE: Eerder in de workshop gebruikte we ``"`` om aan te geven dat een waarde een string is. De backslash (``\``) is echter een special character, waardoor deze verkeerd wordt geïnterpreteerd. Gebruik daarom een enkele qoute ``'`` voor Windows paden. Of ``\\`` met double quotes.
+
* Voer je playbook uit:
+
[source,lang=bash]
----
$ ansible-playbook windows.yml --ask-pass
----
+
* Controleer op je Windows VM het resultaat:
+ 
[source,lang=dos]
----
> type c:\workshop\foo.txt
----

### Task 11.3: Host file bewerken

Als je al eens in Windows de host file aan hebt moeten passen, dan weet je dat dat nog niet zo eenvoudig is. Je moet in Windows de hostfile namelijk als Administrator bewerken. Als je tenmiste het pad nog weet (want die is in Windows ook niet erg eenvoudig).

Ansible heeft voor de hostfile een module: ``win_hosts``. Het bewerken van een hostfile was nog nooit zo makkelijk!

* Voeg toe aan je playbook:
+
[source,role=copypaste]
----
  - name: Add 1.2.3.4 as an A record for www.example.com
    win_hosts:
      state: present
      canonical_name: www.example.com
      ip_address: 1.2.3.4
----
+
* Voer je playbook uit en controleer het resultaat op je Windows VM:
+
[source,lang=dos]
----
> type c:\windows\system32\drivers\etc\hosts
1.2.3.4 www.example.com
> ping www.example.com
Pinging www.example.com [1.2.3.4] with 32 bytes of data:
----

### Task 11.4: Registry aanpassen

In het register van Windows worden instellingen voor allerlei applicaties opgeslagen. Een voorbeeld daarvan is de ``bureaublad achtergrond``. Deze wordt opgeslagen in de ``Registry key``: ``HKEY_CURRENT_USER\Control Panel\Desktop\`` als string (``REG_SZ``) in de key ``WallPaper``.

We gaan een willekeurige wallpaper downloaden en deze installeren als bureaublad achtergrond.

* Voeg toe aan je playbook:
+
[source,role=copypaste]
----
  - name: Download wallpaper to specified path
    win_get_url:
      url: https://images.pexels.com/photos/2536643/pexels-photo-2536643.jpeg
      dest: "{{ WORKAROUND_WALLPAPER }}"

  - name: Ensure the wallpaper is configured
    win_regedit:
      path: HKCU:\Control Panel\Desktop
      name: WallPaper
      data: "{{ WORKAROUND_WALLPAPER }}"
----
+
* Voeg een variable toe voor het pad van de wallpaper: ``path_wallpaper: C:\workshop\wallpaper.jpg``. 
+
TIP: Kijk terug in link:06_NL_role_haproxy[Lab 6 Role - HAProxy] hoe je variablen toevoegt aan je playbook.
+
* Voer het playbook uit en controleer of de achtergrond gewijzigd is.
+
NOTE: Log uit en opnieuw in op je Windows VM om de wijziging actief te maken
+

