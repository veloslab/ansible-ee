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
Playbook with packages argument:

```yaml
- hosts: all
  roles:
    - role: veloslab.base.python_venv
      python_venv_name: veloslab
      python_venv_version: 3.12
      python_venv_packages:
        - requests
        - hvac
```
Playbook with requirements argument:

```yaml
- hosts: all
  roles:
    - role: veloslab.base.python_venv
      python_venv_name: veloslab
      python_venv_version: 3.12
      python_venv_requirements: '/dir/path/requirements.txt'
```

License
-------

BSD

