## Lab 5: Playbook - Apache configureren

In dit lab gaan we Apache configureren met alles wat nodig is om uiteindelijk web pagina's te laten zien. 

### Task 5.1: Apache configuratie

Om er voor te zorgen dat Apache gestart wordt tijdens het booten, moet de Apache service ``enabled`` worden. We zorgen er in dezelfde taak ook meteen voor dat de toestand van de apache service op ``started`` wordt gezet.

Daarnaast configureren we de firewall zodat de services ``http`` en ``https`` open gezet worden. Dit voeren we ``immediate`` uit, maar we slaan de configuratie ook ``permanent`` op. 

TIP: Bekijk ook zeker de documentie van de modules: https://docs.ansible.com/ansible/latest/modules/systemd_module.html en https://docs.ansible.com/ansible/latest/modules/firewalld_module.html zodat je begrijpt wat alle parameters doen.

* Vul het playbook ``webservers.yml`` aan met:

[source,role=copypaste]
----
  - name: "Ensure apache service is enabled and started"
    systemd:
      name: httpd
      enabled: true
      state: started

  - name: "Ensure firewall is configured"
    firewalld:
      service: "{{ WORKAROUND_ITEM }}"
      permanent: yes
      immediate: yes
      state: enabled
    with_items:
      - http
      - https
----

NOTE: In de ``firewall`` task gebruiken we een loop ``with_items`` om een lijst met meerdere services op te geven. Met ``with_items`` wordt de task steeds opnieuw uitgevoerd met een onderdeel van deze lijst. Dit onderdeel wordt dan in de variable ``{{ WORKAROUND_ITEM }}`` gezet. Variablen worden in Ansible genoteerd tussen ``{{ WORKAROUND_EN }}``.  Meer info over loops: https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html

TIP: Ansible werkt met een ``desired state``. Als de service apache nog niet gestart is, zal Ansible er voor zorgen dat deze gestart wordt. We hebben immers de ``state`` ``started`` geconfigueerd. Daarnaast is Ansible ``idempotent``. Dat wil zeggen dat Ansible eerst checkt of het wel nodig is om de task uit te voeren. Ansible controleerd daarom eerst of ``httpd`` al gestart is. Alleen als ``httpd`` niet de status ``started`` heeft wordt de taak uitgevoerd.


### Task 5.2: Webpagina index.html installeren

Om de webservers daadwerkelijk wat te laten zien, maken we een index.html in de default webserver root (``/var/www/html``). Om later onderscheid te kunnen maken tussen web1 en web2 gebruiken we de variable ``{{ WORKAROUND_FQDN }}``. 

* Vul het playbook ``webservers.yml`` aan met:

[source,role=copypaste]
----
  - name: "Ensure index.html is installed"
    copy:
      content: "This is server {{ WORKAROUND_FQDN }}"
      dest: "/var/www/html/index.html"
----

TIP: Ansible haalt ``facts`` op van elk systeem. Deze ``facts`` bevatten waardevolle variablen over de hosts. De module ``setup`` is verantwoordelijk voor deze ``facts``. Met een ``adhoc`` commando kun je deze facts eenvoudig inzien: ``ansible -m setup web1``. Zie https://docs.ansible.com/ansible/latest/modules/setup_module.html

* Start het playbook. Als alles goed is gegaan, zijn de webservers nu klaar voor gebruik!

[source]
----
$ ansible-playbook webservers.yml
----

### Task 5.3: Werking testen

Als het playbook alleen nog maar "ok" meldingen geeft, is het tijd om de werking van de beide webservers te controleren.

[source]
----
PLAY RECAP ********************************************************************************************************************************************************************************************************************
web1                       : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2                       : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
----

* Open via je webbrowser de URL van de eerste webserver: http://{{ ANSIBLE_CLIENT_1 }}.
* Controleer of de hostname in de pagina correct is:

[source]
----
This is server {{ ANSIBLE_CLIENT_1 }}
----

* De tweede webserver: http://{{ ANSIBLE_CLIENT_2 }}:
[source]
----
This is server {{ ANSIBLE_CLIENT_2 }}
----

Volgende Stap: [Lab 6 Role - HA Proxy](06_NL_role_haproxy.md)
