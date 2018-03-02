# Documentation

- [Introduction](#introduction) - what adoc is
- [Inline Documentation](#inline-documentation) - how documentation works
  - [Doc YAML](#yaml-format) - The format for a documentation entry
  - [Python Injection](#python-injection) - how to document in python
  - [Shell Script Injection](#shell-script-injection) - how to document in a shell script
  - [Role Injection](#role-injection) - how to document in an ansible Role
  - [Playbook Injection](#playbook-injection) - how to document in an ansible Playbook
- [Indexes](#indexes) - how the indexes are generated
  - [adoc.yml](#adoc.yml-format) - How to extend index documentation
- [Usage](#usage) - How to use the tool
   

## Introduction

The documentation engine for ansible-ops is adoc.  It leverages the ideas behind the
ansible-doc(1) tool where details are formatted consistently using YAML and then 
converted into readable documentation. This tool has several features:

- handle documentation for all aspects of an ansible instance:
   - module 
   - role 
   - playbook 
   - binary
- display markdown formatted documentation on the commandline
- search by keyword for a module, role, playbook or binary
- rebuild the full documentation index for the instance suitable for GitHub.
- uses an extended ansible-doc format for data storage
- handles several formats:
  - YAML for playbooks and roles via the 'doc:' module
  - python via the DOCUMENTATION and EXAMPLES variables
  - bash scripts via commented doc: YAML
  

## Inline Documentation
Documentation for each item is stored inline.  This means it can be updated atomically
and then rebuilt into the indexes using the rebuild command.  

## YAML format
supports the following extended attributes:

- short_description: sentence suitable for a link or keyword 
- host_scope: the target(s) for this particular item
- description: a string or list of strings describing the item in detail. For instance 
for playbooks, this should be the concise steps that will run
- options: A dict of the options available to be passed to modify item

   - description: a string or list of strings describing the option
   - required: a boolean describing if it must be defined
   - default: a text entry of what is used if not redefined. This option will 
   automagically update from defaults/main.yml or vars: so what is displayed is
   the actual default
   - options: a list of discrete options which may be selected for this option
  
- returns: a list of variables returned, can include text as a description after.
- notes: a list of notes about item, useful to describe use cases 
- examples: a list of strings displayed as examples.

There are also some options not displayed by default, but can be with -f:
- requirements: a list of required dependencies to run this item
- author/authors: a string/list of strings of the authors.
- version_added: this is relative to the version of ansible
- keywords: a list of strings used as part of the keyword search. 

## Python Injection
Putting documentation in python uses the same format as ansible-doc 

```
DOCUMENTATION = '''
---
module: spine_keys
short_description: reads keys from spine
description:
options:
    spine_dir:
        description:
            - path to the spine config
        required: true
    entity:
        description: the entity to look in
    az:
        description: the az to look in
    role:
        description: a role to look in
        required: false
    keys:
        description:
            - a list of keys
    fail_on_missing:
        description: fail if key is missing
        default: True

'''
```

NOTE: for modules .. the module variable will be converted to 'name'

## Shell Script Injection
Putting Documentation into a shell script, just uses the same YAML format which
is commented.  It requires a clean carriage return at the end.
```
# doc:
#   module: spine
#   short_description: run spine on a remote host
#   host_scope: mcp
#   description:
#     - just runs spine and returns the output in msg
#   authors: 
#     - "Branson Matheson, <brmathes@cisco.com>"
#   options:
#     release:
#       description: spine release to use
#       default: latest
#     dryrun:
#       description: use dryrun mode
#       options:
#          - yes
#          - no
#   returns:
#     - changed - if status changes.
#     - msg - spine run output
#   keywords:
#     - spine
#     - spine-mgmt
#   examples:
#     - |
#       spine:
#         dryrun: yes
#         release: latest
```

## Role Injection
Putting documentation into a role, we use the 'doc' module. Some notes:
  
- This module can execute and checks the syntax of the doc: variables, however 
it's not necessary and can be disabled with a 'when: False'
- default variables expressed as Jinja2 expansions will attempt to be expanded
against variables defined in defaults/main.yml.  This means the user will see
the actual data that will be used. (don't put passwords here!)

```
---
- name: documentation
  when: False
  doc:
    short_description: start or append an IPMI console to a screen instance
    host_scope: localhost
    description:
      - Starts screen locally
      - attaches ipmi console if it is activated on far end
      - uses the names passed in the playt
      - generates ucs hostname by inserting ".oob."
    options:
      ucs_host: 
        description: the FQDN to attach to 
        default: "{{ ucs_host }}"
      baudrate:
        description: the default baud rate 
        default: "{{ baudrate }}"
    version_added: 1.9
    author: "Branson Matheson <brmathes@cisco.com>"
    keywords:
      - ipmi
      - console
      - serial
    examples:
      - |
        - name: start serial console on screen
          hosts: all
          serial: 1
          gather_facts: false
          roles:
            - { role: local/ipmi_console }
```

## Playbook Injection
Putting documentation in a playbook is the same as putting it in a role, with 
a few caveats:

- the documentation should be put in as tasks.  This can be in any of:

   - pre_tasks
   - tasks
   - post_tasks

- defaults are expanded from the vars: entry in the playbook if it exists. 

```
#-- update MHVs
- name: apply spine on MHVs
  hosts: mhv
  serial: 1
  max_fail_percentage: 0
  vars:
    reboot: False
    disable: False
    
  pre_tasks:
    - name: doc
      when: False
      doc:
        short_description: run spine on MHVs in an AZ
        host_scope: mcp
        description:
          - put host in maintenance and update the ticket
          - evacuate and disable nova and neutron 
          - set ceph noout if needed
          - run spine
          - reboot if that's set
          - re-enable the node when it's up
          - remove maintenance
        notes:
          - set to run one at a time.
        options:
          disable:
            description: remove host from cluster cleanly 
            default: "{{disable}}"
          reboot:
            description: reboot the host after updates
            default: "{{reboot}}"
        version_added: 1.9
        author: "Branson Matheson <brmathes@cisco.com>"
        examples: 
          - 'ansible-playbook playbooks/spine/az/update_certs.yml'
```

## Indexes
adoc manages all the documentation in the managed directories. It does this by recursing
the directories, building the index .. and then using that index to build the indexes
for those directories.  It's quite simple to run:

```
brmathes@BRMATHES-M-C0SW [stage1.mc] > adoc -R
brmathes@BRMATHES-M-C0SW [stage1.mc] > find roles playbooks bin library -name 'README.md'
roles/ansible/config/keystone/README.md
roles/ansible/config/keystone/ssl/README.md
roles/ansible/config/README.md
roles/ansible/README.md
roles/install/base-passwords/README.md
roles/install/git_tool/README.md
...

```

### adoc.yml Format
Because we can't store all the info concerning what a directory
is for.. we also have 'tree' in 'adoc.yml' in the root of the ansible instance to expand data
for sublinks.  Each entry in the tree has several objects.

```
tree:
  bin:
    title: Binaries
    description:
    - These are scripts that we use in the day-to-day working of
    
 roles/ansible:
    short_description: manage local ansible itself
```

The tree is organized by directory name and each name has these options:

- title: something to replace the directory name to be more concise
- description: a list of items to show inline about the item
- short_description: a quick sentence suitable as a link description
- footer: a list of strings put at the bottom of the file.



## Usage
Running this tool is quite simple.

- adoc with no arguments gives the options:
```
brmathes@BRMATHES-M-C0SW [stage1.mc] > adoc
usage: adoc.py [-h] [-v] [-e] [-H ANSIBLE_HOME] [-k KEYWORD_SEARCH] [-r] [-p]
               [-b] [-l] [-R] [-f]
               [item]

search and manage ansible documentation

positional arguments:
  item                  item to show the docs for

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         Set verbose output.
  -e, --show-errors     show the missing doc entries
  -H ANSIBLE_HOME, --ansible-home ANSIBLE_HOME
                        path to ANSIBLE_HOME.
  -k KEYWORD_SEARCH, --keyword-search KEYWORD_SEARCH
                        search keywords.
  -r, --roles-filter    Filter return for roles.
  -p, --playbooks-filter
                        Filter return for playbooks.
  -b, --bin-filter      Filter return for binaries.
  -l, --lib-filter      Filter return for libraries.
  -R, --rebuild-markdown
                        rebuild inline markdown files
  -f, --full-details    show more details about this entry
```

- adoc with a single word.. will find all occurances of that word in the names of all searchable items

```
brmathes@BRMATHES-M-C0SW [stage1.mc] > adoc console

 bin/console
    - creates a tmpfile with hosts as noted on the command line
    - uses playbooks/operations/cimc/get_sol_console.yml to startup consoles
    - calls screen -r -d to attach to them locally
  Options:
    - {hostname} - shortform hostname (eg .. mhv1)
  Notes:
  - does NOT close the screen session if you exit w/o terminating.
  Examples

  |  console mhv1


 roles/local/ipmi_console
  Hosts: localhost
    - Starts screen locally
    - attaches ipmi console if it is activated on far end
    - uses the names passed in the playt
    - generates ucs hostname by inserting ".oob."
  Options:
    - baudrate - the default baud rate
        - default: "115200"
    - ucs_host - the FQDN to attach to
        - default: "{{ inventory_hostname.split('.')[0] }}.oob.{{ inventory_hostname.split('.')[1:] | join('.') }}"
  Examples

  |  - name: start serial console on screen
  |    hosts: all
  |    serial: 1
  |    gather_facts: false
  |    roles:
  |      - { role: local/ipmi_console }
```

- can also constrain the search by adding a keyword
```
brmathes@BRMATHES-M-C0SW [stage1.mc] > adoc maintenance -p -k az

 playbooks/operations/az/maintenance_remove
  Hosts: mcp:mhv CIMC
    - removes maintenance in zabbix for the entire AZ
  Options: None
  Notes:
  - requires AZ and TICKET ontext to be set
  - will only remove maintenance for THIS context
  Examples

  |  ansible-playbook operations/az/maintenace_remove.yml


 playbooks/operations/az/maintenance_set
  Hosts: mcp:mhv CIMC
    - creates a maintenance in zabbix for the entire AZ
  Options: None
  Notes:
  - requires AZ and TICKET ontext to be set
  - will only add maintenance for THIS context
  Examples

  |  ansible-playbook operations/az/maintenace_set.yml
```

- adoc with just -k .. performs a keyword search 
```
brmathes@BRMATHES-M-C0SW [stage1.mc] > adoc -k console
keyword: console
  roles/local/ipmi_console - start or append an IPMI console to a screen instance
  bin/console - grab the SOL consoles on remote hosts
```
