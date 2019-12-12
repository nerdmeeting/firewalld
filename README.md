Ansible Role: Firewalld
=========

Role to mange some Firewalld features

Requirements
------------

* Ansible >= 2.4
* Linux Distribution
    * RedHat Family
        * CentOS
            * 7
        * Note: Other versions are likely to work but have not been tested.

Role Variables
--------------

The following variables will change the behavior of this role (default values
are shown below):

```yaml
# Allow/disallow inbound remote access to port 80
allow_remote_port_80: false

# Allow/disallow inbound remote access to port 8080
allow_remote_port_8080: false
```


Example Playbook
----------------

    - hosts: web
      roles:
         - { role: nerdmeeting.firewalld }

*Inside `vars/main.yml`*:

    allow_remote_port_80: true 

License
-------

MIT / BSD

Author Information
------------------

Role created in 2019 by [Nathan Bills](https://nerdmeeting.com)
