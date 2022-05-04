Role Name
=========

Deploy Consul Server or Client

Requirements
------------

None

Role Variables
--------------

`consul_version` - String, version of consul that should be installed. Default is '*'

`consul_server` - Boolean, determines if consul agent should be set up as server or client

`consul_datacenter` - String, Name of datacenter

`consul_encrypt` - String, Encryption key for gossip

`consul_retry_join` - List of consul servers

`consul_ca_cert` - String, content of ca certificate to use for tls

`consul_bootstrap_expect` - Int, number of consul servers to expect. Only required when consul_server is true

Following variables should be set for hosts that will be consul servers, these should be host variables. Only required on consul server hosts

`consul_server_cert` - String, content of server certificate

`consul_server_key` - String, content of server key

Dependencies
------------

None

Example Playbook
----------------

Exmple of deploying 

```yaml
  ---
  - hosts: all
    become: true
    vars:
      consul_datacenter: 'dc1'
      consul_encrypt: "{{ lookup('hashi_vault', 'secret=secret/data/consul/encryption_key')['key']}}"
      consul_server: true
      consul_retry_join: ["consul-1.domain", "consul-2.domain", "consul-3.domain"]
      consul_bootstrap_expect: 3
  
    tasks:
      - name: Deploy Consul Server
        include_role:
          name: veloslab.hashicorp.consul
        vars:
          consul_server_tls: "{{ lookup('hashi_vault', 'secret=secret/data/consul/{{ ansible_hostname }}')}}"
          consul_ca_cert: "{{ consul_server_tls['ca'] }}"
          consul_server_cert: "{{ consul_server_tls['certificate'] }}"
          consul_server_key: "{{ consul_server_tls['private_key'] }}"                                                                        
```


License
-------

BSD
