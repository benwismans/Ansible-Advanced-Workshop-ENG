## Introduction

Welcome to the Ansible Advanced Workshop. In this workshop we will set up a load balancer with 2 web servers. Also, we will create an Ansible playbook to automatically update this environment without interruptions.

### Requirements
For this workshop, you will need:

* Laptop with a SSH client (for example putty: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
* A Linux Server with SSH connection
* Local user on the VM
* Root permissions on the VM

## Execution
- All actions will be done from the user's home directory on the 1st VM (unless otherwise mentioned)
- Instructions for commands will be preceded with a promot sign ($) and in a text block. This sign is for of the prompt and does not belong to the actual command. (The command in the example is ``ls``):

  ``$ ls``
  
- The output is alsow shown in a text block. For example:
  ```
  user@vm01:~ $ ls -l
  total 0
  ..
  .
  ```

Next step: [Lab 2 Inventory file](02_NL_inventory.md)

