Role Name
=========

JumpCloud free tier limits JumpCloud Agent usage to only 10 devices. Agent makes it easy to manage user's ssh access. Instead of relying on JC's Agent, you can set up SSSD to allow  user access

Requirements
------------

Only tested on Debian/Ubuntu

Role Variables
--------------
Variables required for role:

    jumpcloud_sudoers - Default {}, where key is user/group and value is true/false. Value determines if password is required for sudo. 
    By default, no group or user is added to sudoers
    
    jumpcloud_organization - JumpCloud Organization ID, can be found in ldap settings
    
    jumpcloud_bind_user - Bind User
    
    jumpcloud_bind_password - Password for Bind User

Dependencies
------------

None

Example Playbooks
----------------

Example below shows adding a group (group name must be prepended with %) and user to sudoers. 
Password is required for mygroup to sudo 
```yaml
- hosts: all
  vars:
    jumpcloud_sudoers: {'%mygroup': true, 'myuser': false}
    jumpcloud_bind_user: 'buser'
    jumpcloud_bind_password: 'password'
    jumpcloud_organization: '1123a1b3342'

  roles:
    - veloslab.jumpcloud.sssd
```

Below, no group or user will be added to sudoers, using default value
```yaml
- hosts: all
  vars:
    jumpcloud_bind_user: 'buser'
    jumpcloud_bind_password: 'password'
    jumpcloud_organization: '1123a1b3342'

  roles:
    - veloslab.jumpcloud.sssd
```

License
-------

BSD
