# Lab 7: Ansible Vault

Ansible Vault is a function within ansible that can encrypt your senstive data, like passwords, private kets or SSL certificates. Ansible Vault makes it possible to still put this data in a ``SCM`` (Source Code Management; like github).

Vault variables get decrpyted by Ansible automaticcaly, if you provide the ``vault password``. A vault variable that is encrypted always starts with ``$ANSIBLE_VAULT;1.1;AES256``.


## Task 7.1: Encrpyting files

**Tip:** ansible-vault uses the editor ``vi`` by default. If you would rather use ``nano``, then you can adjust the editor by editing the environment variable ``EDITOR`` with: ``/bin/nano``. You can use the command: ``export EDITOR=/bin/nano``.

* Create a new file:

  ``$ ansible-vault create foo``

* Come up with your own password:

  ```
  New Vault password:
  Confirm New Vault password:
  ```

* Put a secret message in the file and save it (in ``vi`` that's ``:wq``).
* Inspect the file (it's encrypted):

  ``$ cat foo``
  
  ```
  $ANSIBLE_VAULT;1.1;AES256
  36323433633666373164336366383438613964306431646537643863343762663465376265326337
  6335663530313034653733323431376236316637643536610a323537336233373139363538383438
  65633532333866356339333238653964633938363661353331373237343366306239313632623935
  3738363065653864660a326666316531643837366161656531366239376338343534336230613832
  6563
  ```

## Task 7.2: Editing an encrypted file

* Edit the file:

  ``$ ansible-vault edit foo``

* Ansible asks the password. 

  ```
  Vault password:
  ```

* Make a change and save again.


## Task 7.3: Looking at an encrypted file

* View the file:

  ``$ ansible-vault view foo``

* Ansible asks the password. 

  ```
  Vault password:
  <decrpyted data>
  ```

## Task 7.4: Changing the password of an encrypted file

* Adjust the password:

  ``$ ansible-vault rekey foo``
  
   ```
   Vault password:
   New Vault password:
   Confirm New Vault password:
   Rekey successful
   ```

## Task 7.5: Using an encrypted file in your playbook

Ansible will automatically recognize and encrypted file and will try to decrypt it. This works for all modules. You need to provide the ``vault password`` when running the ``ansible-playbook`` command. The parameter is ``--ask-vault-pass``.

* Add to your playbook ``workshop.yml``:

  ```
    - name: "Ensure foo is copied and decrypted"
      template:
        src: foo
        dest: /home/userXX/foo 
  ```

* Run the playbook (make sure you add the parameter ``--ask-vault-pass``):

  ``$ ansible-playbook workshop.yml --ask-vault-pass``

* Check if the file is decrypted on your client system:

  ``$ cat foo``
  
  ```
  $ cat foo 
  <decrypted data>
  ```

**Tip:** A good use case for Vault is when using SSL certificates. An SSL certificate always contains a public and a private key. The private key needs to be protetcted. It's not wise to put the private key in plain text in an Ansible script. That's why you should encrypt it with Ansible Vault.

  
Next Step: [Lab 8 Role - Create a Role ](08_NL_role_create.md)
