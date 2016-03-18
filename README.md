Virtual Router Role for EOS
===========================

The arista.eos-virtual-router role creates an abstraction for common EOS
virtual router configuration. This means that you do not need to write any
ansible tasks. Simply create an object that matches the requirements below
and this role will ingest that object and perform the necessary configuration.

This role is used to configure VARP-related configuration. There is
an assumption that the VARP configuration will be placed on Vlan interfaces.
This role will take care of creating those Vlans and configuring the IP address
for that Vlan interface if they do not already exist.

Installation
------------

```
ansible-galaxy install arista.eos-virtual-router
```


Requirements
------------

Requires an SSH connection for connectivity to your Arista device. You can use
any of the built-in eos connection variables, or the convenience ``provider``
dictionary.

Role Variables
--------------

The tasks in this role are driven by the ``virtual_mac_addr``,
``varp_interfaces`` objects described below:

**virtual_mac_addr**

|       Key        |  Type  | Notes                                    |
| :--------------: | :----: | ---------------------------------------- |
| virtual_mac_addr | string | The MAC address to assign as the virtual-router mac address. This value must be formatted like aa:bb:cc:dd:ee:ff |


**varp_interfaces** (list) each entry contains the following keys:

|            Key | Type                      | Notes                                    |
| -------------: | ------------------------- | ---------------------------------------- |
|         vlanid | string (required)         | The Vlanid where the Varp configuration will be placed. |
|           name | string                    | The name given to the Vlan.              |
|    description | string                    | The description placed on the Vlan interface. |
|         enable | boolean: true*, false     | Enable or disable the Vlan and Vlan interface. |
| interface_addr | string                    | The IP address assigned to the Vlan interface. Of the form, X.X.X.X/Y |
|  virtual_addrs | list                      | A list of IP addresses that will be shared among the pair of switches in the virtual router configuration. Pre-configured IP addresses not in the list will be removed from the Vlan interface. |
|          state | choices: present*, absent | Set the state for the route configuration. |


```
Note: Asterisk (*) denotes the default value if none specified
```


Dependencies
------------

The eos-virtual-router role is built on modules included in the core Ansible code.
These modules were added in ansible version 2.1

- Ansible 2.1.0

Example Playbook
----------------

The following example will use the arista.eos-virtual-router role to configure
the virtual-router mac address as well as Vlan interfaces with shared IP addresses.
We'll create a ``hosts`` files with our switches, then a corresponding
``host_vars`` file for each switch and then a simple playbook which only
references the eos-virtual-router role. By including the role, we automatically
get access to all of the tasks to configure these EOS features. What's nice
about this is that if you have a host without any corresponding configuration,
the tasks will be skipped without any issue.

For the example below, let's assume MLAG is already configured. Check out the
arista.eos-mlag role to quickly configure mlag.

Sample hosts file:

    [leafs]
    leaf1.example.com
    leaf2.example.com


Sample host_vars/leaf1.example.com

    virtual_mac_addr: "00:1c:73:00:00:99"
    
    varp_interfaces:
      - vlanid: 1000
        name: Varp_Vlan1000
        state: absent
      - vlanid: 1001
        name: Varp_Vlan1001
        description: My Vlan1001
        enable: true
        interface_addr: 192.168.1.3/24
        virtual_addrs:
          - 192.168.1.1
          - 192.168.11.1
      - vlanid: 1002
        name: Varp_Vlan1002
        description: My Vlan1002
        enable: true
        interface_addr: 192.168.2.3/24
        virtual_addrs:
          - 192.168.2.1
          - 192.168.12.1


Sample host_vars/leaf2.example.com

    virtual_mac_addr: "00:1c:73:00:00:99"
    
    varp_interfaces:
      - vlanid: 1001
        name: Varp_Vlan1001
        description: My Vlan1001
        enable: true
        interface_addr: 192.168.1.4/24
        virtual_addrs:
          - 192.168.1.1
          - 192.168.11.1
      - vlanid: 1002
        name: Varp_Vlan1002
        description: My Vlan1002
        enable: true
        interface_addr: 192.168.2.4/24
        virtual_addrs:
          - 192.168.2.1
          - 192.168.12.1


A simple playbook, leaf.yml

    - hosts: leafs
      roles:
        - arista.eos-virtual-router

Then run with:

    ansible-playbook -i hosts leaf.yml

License
-------

Copyright (c) 2015, Arista Networks EOS+
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

* Neither the name of Arista nor the names of its
  contributors may be used to endorse or promote products derived from
  this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

Author Information
------------------

Please raise any issues using our GitHub repo or email us at ansible-dev@arista.com

[quickstart]: http://ansible-eos.readthedocs.org/en/latest/quickstart.html