Alerta Heartbeat
=========

Ansible role that creates a daemon using supervisor that sends a heartbeat to alerta

Requirements
------------
Only tested on Ubuntu/Debian machines

Role Variables
--------------
Here are defaults variables for alert set up  (see `defaults/main.yml`):

    alerta_heartbeat_timeout: 120
    alerta_heartbeat_environment: production
    alerta_heartbeat_service: system
    alerta_heartbeat_origin: "{{ ansible_hostname }}"

The following variables must be defined
    
    alerta_heartbeat_api_key - API Key
    alerta_heartbeat_domain - API Endpoint Ex. https://domain.com/api

Dependencies
------------
None

Example Playbook
----------------
Playbook:

```yaml
- hosts: all
  vars:
    alerta_heartbeat_api_Key: ABCD1234FGHJ
    alerta_heartbeat_domain: 'https://domain.com/api/'

  roles:
    - veloslab.alerta.heartbeat
```

License
-------

BSD

