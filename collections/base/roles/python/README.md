Alerta Heartbeat
=========

Ansible role that install base python

Requirements
------------
Only tested on Ubuntu/Debian machines

Role Variables
--------------
You need to set the following variable

`python_version` - Python version you want installed

Dependencies
------------
None

Example Playbook
----------------
Playbook:

```yaml
- hosts: all
  vars:
    python_version: 3.10

  roles:
    - veloslab.base.python
```

License
-------

BSD

