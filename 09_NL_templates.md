## Lab 9: Role - Templates 

For configfiles it is best practice to use a ``template``. For templates the ``jinja2`` format is used. This ia also used within python. See: http://jinja.pocoo.org/.

In best pracices, every value in a configule is put in a variable. The default settings of the configfile then are defined in ``defaults/main.yml``.

### Task 9.1: Default variables 

We will rebuild the role of lab 8 to a template.

* Go to the directory of the role ``chrony``:
```
$ cd ~/.ansible/roles/chrony
```
* Add to the file ``defaults/main.yml``:
```
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
```

**[NOTE]**
Note that the ``ntp-servers`` is a list. The variable ``ntp_rtcsync`` is a ``boolean``. 

### Task 9.2: Template 

* First delete the file ``files/chrony.conf``. We will adjust this to a template.
```
$ rm files/chrony.conf
```
* Create a new file ``templates/chrony.conf.j2`` en vul deze met:
```
# Record the rate at which the system clock gains/losses time.
driftfile {{ ntp_driftfile }}

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep {{ ntp_makestep }}

# Specify directory for log files.
logdir {{ ntp_logdir }}
```
* For ntp server we need a ``for`` loop. Add:
```
{% for server in ntp_servers %}
server {{ server }} iburst
{% endfor %}
```
* For ``rtcsync`` we use an ``if``. Add:
```
{% if ntp_rtcsync %}
rtcsync
{% endif %}
```
* In the ``tasks/main.yml`` we need to change the ``copy`` module to the ``template`` module. 

* Change the filename in the ``src`` from ``chrony.conf`` to ``chrony.conf.j2``

* Run the playbook and check on a client if the file ``/etc/chrony.conf`` was changed.
```
$ ssh {{ ANSIBLE_CLIENT_1 }}
$ cat /etc/chrony.conf
```
```
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
...
...
```


### Task 9.3: Role variables in a playbook

* Add the variables in the playbook:
```
ntp_servers:
- 0.europe.pool.ntp.org
- 1.europe.pool.ntp.org
- 2.europe.pool.ntp.org
- 3.europe.pool.ntp.org

ntp_rtcsync: no
```

* Run the playbook and check for changes.
```
$ ssh {{ ANSIBLE_CLIENT_1 }}
$ cat /etc/chrony.conf
```
```
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.europe.pool.ntp.org iburst
server 1.europe.pool.ntp.org iburst
server 2.europe.pool.ntp.org iburst
server 3.europe.pool.ntp.org iburst
...
...
```


Next Step: [Extra Assignment 1 - Rolling Upgrade](E1_NL_rolling_updates.md)
