# role-bind

Ansible role to install and configure ISC BIND on RHEL/CentOS 7.

The role is fully generic and:

 - it installs the latest or a specific version of BIND, available through the system repositories;
 - is capable to write any BIND configuration using a [DLS written in YAML](BIND-DSL);
 - can create `master` and `reverse lookup` zone files for pre set records;
 - it supports `SOA`, `NS`, `MX`, `A`, `CNAME`, `PTR`, `TEXT` and `SRV` records.

The role is quite generic and support most if not all the feature of BIND.
For example, it can be used to configure a local forwarder, a local cache or a DNS master server with reverse lookup or a slave server.

The role does not:

 - take care of opening ports on the local firewalls.
 - support IPv6.

## Requirements

The role requires RHEL/CentOS 7 to work.

The role comes with a custom set of Python filters, [bind_filters.py](library/filters/bind_filters.py), used by the role to build the BIND configuration.
The python file must be copied in the Ansible home path `${ANSIBLE_HOME}` or inside the library path defined by the variable [filter_plugins](https://docs.ansible.com/ansible/latest/intro_configuration.html#filter-plugins) of the [ansible.cfg](https://docs.ansible.com/ansible/latest/intro_configuration.html) configuration file. If the file is missing, Ansible will complain throwing a "`no filter named xxx`" error.

## Role Variables

Here is only a quick summary of some of the configuration variables.

The variables are fully documented in the [default configuration](defaults/main.yml) file, including their default values and some examples.
The default values mirror the default BIND configuration for the distribution where it is installed.

The following variables are used directly to build the BIND main configuration file and correspond to the main statement of the file:
 - `bind_acls`
 - `bind_options`
 - `bind_logging`
 - `bind_includes`
 - `bind_zones`
 - `bind_masters`
 - `bind_server`
 - `bind_view`
 - `bind_statistics_channels`
 - `bind_controls`

These variables are used to fill the zone files managed by the role, their content is fully described by examples in the [default configuration](defaults/main.yml) file:
 - `bind_zones_master`
 - `bind_zones_master_reverse`

The role can be used with the default values but additional options can still be specified using the following variables:
 - `bind_options_additional`
 - `bind_includes_additional`
 - `bind_zones_additional`
 - `bind_logging_additional`

## Example Playbook

The default configuration mirrors the default `named.conf` values of the distribution where it is installed. Therefore it is not necessary to specify any additional values for the role to work.

```Yaml
- hosts: servers
  roles:
     - role: role-bind
```

## BIND DSL

A simple YAML DLS (domain specific language) is used to represent the many options of the BIND configuration file inside Ansible. The format is composed of 7 YAML attributes:

 - comment
 - name
 - value
 - options
 - list
 - inline
 - block

The structure defined in the configuration variables will be parsed top to bottom, so if a comment is defined before a configuration parameter, it will apper before that in the configuration file.

The  [default configuration](defaults/main.yml) file contains the default BIND configuration defined in this DSL and can be used as a good reference for it.

### comment

**Value**: string

**Description**: A simple comment that will be written in the configuration.

**Example**:

Ansible configuration:

```Yaml
bind_options:
 - comment: "Runtime"
```

named.conf

```
options {
    // Runtime
};
```

### name

**Value**: string

**Description**: The name of a BIND statement.

**Example**:

Ansible configuration:

```Yaml
bind_options:
 - { name: pid-file, value: '"/var/named/named.pid"' }
```

named.conf

```
options {
    pid-file "/var/named/named.pid";
};
```

### value

**Value**: string

**Description**: the value of a BIND statement. This is used when the value is a simple element and not a compound one. Be aware that quotes are not added automatically, to cope with the different type of values that the configuration file supports, and for string they must be added in the YAML value.

**Example**:

Ansible configuration:

```Yaml
bind_options:
 # Note the presence of double quotes in the "value" attribute
 - { name: pid-file, value: '"/var/named/named.pid"' }
```

named.conf

```
options {
    pid-file "/var/named/named.pid";
};
```

### options

**Value**: list of strings.

**Description**: List of options (statements either present or absent) separated by a whitespace that are added after the statement value but before the statement block.

**Example**:

Ansible configuration:

```Yaml
bind_zones:
 - name: zone
   value: '"."'
   options: ['IN']
   block:
    - { name: type, value: hint }
    - { name: file, value: '"named.ca"' }
```

named.conf

```
zone "." IN {
    type hint;
    file "named.ca";
};
```

### list

**Value**: list of strings.

**Description**: A list of values separated by a semicolon. As with the [value](#value) attribute the quotes are not added automatically if needed.

**Example**:

Ansible configuration:

```Yaml
bind_acls:
 - name: acl
   value: '"default_networks"'
   block:
    - list:
       - 10.10.0/24
       - 10.20.0/24
       - 10.30.0/24
```

named.conf

```
acl "default_networks" {
    10.10.0/24;
    10.20.0/24;
    10.30.0/24;
};
```

### inline

**Value**: list of dictionaries. Each dictionary is a nested BIND DSL (recursive definition).

**Description**: This is an additional configuration that sits inline between the name of the BIND statement and its value. Every inline value will be written after the statement name and separated by a whitespace.

**Example**:

Ansible configuration:

```Yaml
bind_options:
 - name: listen-on
   inline:
    - { name: port, value: "53" }
   block:
    - list: "192.168.0.1"
```

named.conf

```
options {
    listen-on port 53 {
        192.168.0.1;
    };
};
```

### block

**Value**: list of dictionaries. Each dictionary is a nested BIND DSL (recursive definition).

**Description**: The block attribute encloses a nested set of the DSL into curly braces.

**Example**:

Ansible configuration:

```Yaml
bind_options:
 - name: listen-on
   inline:
    - { name: port, value: "53" }
   block:
    - list: "192.168.0.1"
```

named.conf

```
options {
    listen-on port 53 {
        192.168.0.1;
    };
};
```

## License

MIT

## Author Information

Fabrizio Colonna (@ColOfAbRiX)

## Contributors

Issues, feature requests, ideas, suggestions, etc. are appreciated and can be posted in the Issues section.

Pull requests are also very welcome. Please create a topic branch for your proposed changes. If you don't, this will create conflicts in your fork after the merge.
