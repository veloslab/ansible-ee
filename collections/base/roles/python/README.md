Base Python
=========

Ansible role that installs python

Requirements
------------
Only tested on Ubuntu/Debian machines

Role Variables
--------------

The following variables must be defined

`python_version` - Version of Python that needs to be installed

Dependencies
------------
None

Example Playbook
----------------
Playbook:

```yaml
- hosts: all
  vars:
    python_version: '3.6'

  roles:
    - veloslab.base.python
```

License
-------

BSD

