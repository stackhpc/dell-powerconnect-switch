Dell PowerConnect Switch
========================

This role configures Dell PowerConnect switches using the
[expect](http://docs.ansible.com/ansible/latest/modules/expect_module.html)
Ansible module.

This role will install the python `expect` package to the system site packages
on the local machine.

Requirements
------------

The switches should be configured to allow SSH access.

Role Variables
--------------

`dell_powerconnect_switch_provider` is authentication provider information,
similar to the `provider` argument to the `dellos` modules. It should be a dict
containing the following fields:

- `host`: the host or IP address of the switch.
- `username`: the username with which to access the switch via SSH.
- `auth_pass`: the password with which to authenticate.

`dell_powerconnect_switch_config` is a list of configuration lines to apply to
the switch, and defaults to an empty list.

`dell_powerconnect_switch_interface_config` contains interface configuration.
It is a dict mapping switch interface names to configuration dicts. Each dict
may contain the following items:

- `description` - a description to apply to the interface.
- `name` - a name to apply to the vlan interface, if you're configuring a vlan.
- `config` - a list of per-interface configuration.

`dell_powerconnect_switch_write_memory` is a boolean flag, which when set to true, 
will save the switch's running configuration to the startup configuration file, 
after the role applies its configuration. This will allow the configuration to 
persist after a restart or power failure. By default, this option is set to false. 

`dell_powerconnect_switch_write_command` is the command which is run when the flag 
dell_powerconnect_switch_write_memory is set to true. The default command is 
"write memory". 

Dependencies
------------

None

Example Playbook
----------------

The following playbook configures hosts in the `dell-powerconnect-switches`
group.  It assumes host variables for each switch holding the host, username
and passwords.  It applies global configuration for LLDP, enables two 10G
ethernet interfaces as switchports, and saves the configuration changes to
memory. 

    ---
    - name: Ensure Dell PowerConnect switches are configured
      hosts: dell-powerconnect-switches
      gather_facts: no
      roles:
        - role: dell-powerconnect-switch
          dell_powerconnect_switch_write_memory: yes
          dell_powerconnect_switch_provider:
            host: "{{ switch_host }}"
            username: "{{ switch_user }}"
            password: "{{ switch_password }}"
            transport: cli
            authorize: yes
            auth_pass: "{{ switch_auth_pass }}"
          dell_powerconnect_switch_config:
            - "protocol lldp"
            - " advertise dot3-tlv max-frame-size"
            - " advertise management-tlv management-address system-description system-name"
            - " advertise interface-port-desc"
            - " no disable"
            - " exit"
          dell_powerconnect_switch_interface_config:
            Te1/1/1:
              description: server-1
              config:
                - "no shutdown"
                - "switchport"
            Te1/1/2:
              description: server-2
              config:
                - "no shutdown"
                - "switchport"
            "vlan 1234":
              name: "mytestvlan"
              config:
                - "ip address 192.168.1.254 255.255.255.0"

Author Information
------------------

- Mark Goddard (<mark@stackhpc.com>)
