## Extra - Rolling updates

Met Ansible is het mogelijk om ``Rolling updates`` uit te voeren. Webservers achter een loadbalancer kunnen 1-voor-1 geüpdatet worden. Tijdens de update kan de webserver uit de loadbalancer gehaald worden. Pas als de webserver weer volledig up-and-running is (bijvoorbeeld na een reboot), wordt de volgende webserver uit de loadbalancer gehaald om te updaten. Het is daarbij zelfs mogelijk om een HTTP request te doen, om er zeker van te zijn dat de webserver functioneert.

TIP: In dit lab zul je delen zelf moeten invullen. Kijk terug in eerdere labs hoe de onderdelen van het playbook gemaakt zijn.

* Maak een nieuw playbook ``update.yml`` voor de host groep ``webservers``.
* Zorg dat ``become`` is geconfigureerd.
* Maak een task aan om ``status.html`` in /var/www/html/ te zetten. Gebruik de copy module (zie ``Task 5.2: Webpagina index.html installeren``)
* Zet de volgende content in de copy task:
+
[source,role=copypaste]
----
Status OK
----

* Voeg toe aan je playbook:
+
[source,role=copypaste]
----
  - name: "Check status of webserver before proceed"
    become: false
    local_action:
      module: uri
      url: "http://{{ WORKAROUND_HOST }}/status.html"
      status_code: 200
----
+
TIP: De bovenstaande taak wordt onder de speciale module ``local_action`` uitgevoerd. Het opvragen van de URL wordt dus vanuit de bastion server uitgevoerd. 

* Voeg toe aan je playbook:
+
[source,role=copypaste]
----
  - name: "Ensure the webserver is disabled in the backend pool"
    haproxy:
      state: disabled
      host: '{{ WORKAROUND_INVENTORY_HOSTNAME }}'
      backend: habackend
      socket: /var/lib/haproxy/stats
    delegate_to: lb
----
+
TIP: De bovenstaande taak wordt gedelegeerd naar de loadbalancer groep ``lb``. HAProxy draait natuurlijk op de loadbalancer en niet op de webserver waar de taak anders op uitgevoerd zou worden.

* Voeg een task toe die ``index.html`` overschrijft met de volgende content (copy module):
+
[source,role=copypaste]
----
This is server {{ WORKAROUND_FQDN }} and has version 1.0
----

* Voeg een task toe met de ``reboot`` module. Er zijn geen extra parameters vereist.
* Voeg de volgende task toe:
+
[source,role=copypaste]
----
  - name: "Check status of webserver before proceed"
    become: false
    local_action:
      module: uri
      url: "http://{{ WORKAROUND_HOST }}/status.html"
      status_code: 200
      return_content: yes
    retries: 10
    delay: 2
    register: result
    until: result.content.find('Status OK') != -1
----
* Voeg een kopie van de taak ``Ensure the webserver is disabled in the backend pool`` toe en zet de ``state`` op ``enabled``.
* Zet de batch size voor het playbook op 1. Zoek in de handleiding op hoe dat precies werkt. Zie: https://docs.ansible.com/ansible/latest/user_guide/playbooks_delegation.html.

TIP: Door de batch size op 1 te zetten wordt het playbook maximaal op 1 webserver tegelijk uitgevoerd. Dit zorgt er voor dat er tijdens de rolling upate altijd een webserver beschikbaar is in de load balancer.

* Open het adres van de loadbalancer in je webbrowser (http://{{ ANSIBLE_CLIENT_3 }}). Ververs een paar keer de pagina, zodat je zeker weet dat beide webservers beschikbaar zijn.
* Voer het playbook ``update.yml`` uit en ververs regelmatig de pagina van de loadbalancer. Als alles goed is gegaan, blijft de pagina bereikbaar tijdens het updaten.
