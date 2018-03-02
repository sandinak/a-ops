# Architecture

# Ansible
This kit leverages and relies upon the existing production ansible-inventory.  It utilizes several features of ansible to allow for streamlined operations with minimal interruption/impact to operations while working issues.

## Playbooks
Playbooks are broken into discrete directories which denote the scope of the work that the playbook will impact.

* install - tools used as part of build and installation
* operations - discrete common operations we use to modify an AZ
* procedures - standard procedures we use in day-to-ay
* runbooks - documented procedures translated from runbooks
* tests - discrete tests to be used as part of operations and build

## Roles
Roles are scoped by the areas they impact:

* ansible - tools for testing ansible configuration
* install - tools which may used during installation
* metacloud - specifically dealing with integrations around Metacloud operations
* metapod - dealing with the 'system' level operations which occur on pods
* openstack - deals with OpenStack operations

## Ansible Libraries
Ansible Libraries have been written to handle integration between the systems and codebase where appropriate.  There are also some efficiencies built into these tools where ansible looping does not make sense or is too slow.

## Ops Environment Variables
- OPS_HOME - location of the ansible-ops checkout.
- AI - location of the base of ansible inventory .. and where we search for AZ's
- ENTITY - the entity name of the environment ( typically the TLD for a set of AZs )
- AZ - the name of the AZ with model ( so AZ1.pv .. vs AZ1 )
- MC_TXT - what can be stuffed into the prompt for context
- HOST_LINKS - directory where hostnames are linked to scripts that expand them into FQDN for ssh

## Ansible Environment Variables
- ANSIBLE_INVENTORY - the location of the inventory to be used.  In our context this is a sub-directory of the ansible-inventory git repo and tied to a specific AZ
- ANSIBLE\_VAULT_PASSWORD - for our purposes, this is the location of a script which returns a password which is valid for the context set by the user.
- ANSIBLE_HOME - the location of the ansible code base.  For our purposes this will be defined as the home for ansible-ops repo, however we have shortcuts enabled for ansible-systems as well.

# Password Management
We use the pass(1) program to store passwords locally. This may be superseeded with a central password
store for common passwords, however there will always be a need for locally stored passwords for
specific services that are tied to the user rather than the AZ. ansible-ops then uses
these stored passwords in several ways:

- ansible-vault - When a user sets a context for the shell, it will define an environment variable AZ. This is then used by the ansible_vault_password script (via the ANSIBLE_VAULT_PASSWORD_FILE environment variable)  to call pass(1) on the ansible vault password for that AZ and return it to the ansible process.
- lookups - For passwords used inside playbooks, there is a simple way for ansible to request the password using a lookup

```
  "{{lookup('pipe','pass {password_name}')}}"
```

So for instance to lookup zabbix user/pass:

```
  login_user="{{lookup('pipe','pass mc_services/zabbix/user_name')}}"
  login_password="{{lookup('pipe','pass mc_services/zabbix/user_password')}}"
```


