# Ansible-ops
_Version_ : 2.0
_Released_: Wed Feb 15 15:48:53 EST 2017

This is a configured ansible instance that works with exisiting Metacloud ansible-inventory to make it easier to perform typical operations.  The objective is to provide a clean framework for extending operations.

# TOC
- [Installation](docs/INSTALLATION.md) - installation guide
- [Getting Started](docs/GETTING_STARTED.md) - walkthrough to get started
- [Release Notes](docs/RELEASE_NOTES.md) - info about the current installation
- [Usage](docs/USAGE.md) - How to use these tools
    - [Binaries](bin/README.md) - binary tools to use from the command line
    - [Playbooks](playbooks/README.md) - the different playbooks available
    - [Library](library/README.md) - Library modules available for tooling playbooks
    - [Roles](roles/README.md) - roles available for tooling your playbooks
    - [Plugins](plugins/README.md) - plugins to extend ansible capability
- Development Tools
    - [Writing New Playbooks](docs/NEW_PLAYBOOKS.md) - how to write new books
    - [Architecture](docs/ARCHITECTURE.md) - how this kit is designed
    - [Documentation](docs/DOCUMENTATION.md) - how to document tools
    - [Requirements](docs/REQUIREMENTS.md) - the basis for tooling and systems

## Features
_Strong Documentation Tools_

Ansible ops includes the 'adoc' tool which walks your a-ops install tree and uses titles
and keywords to quickly find and display the tools available for your needs.

Search for playbooks(-p) dealing with 'ldap'

```bash
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > adoc -p ldap

 playbooks/tests/openstack/keystone/ldap
  Hosts: mcp
    - test for proper config in ansible
    - test for SSL setup on an MCP
    - test LDAP with ansible keystone domains
  Options: None
  Notes:
    - requires AZ context
  Examples

  |  ansible-playbook tests/openstack/keystone/ldap.yml
```

Search by keyword for anything dealing with 'ldap'

```
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > adoc -k ldap
playbooks/az_specific/sprt/deploy_ca_certs - deploy new certificates to an AZ
playbooks/tests/openstack/keystone/ldap - test keystone LDAP configurtation w/o applying it
roles/openstack/keystone/ldap - Perform tests of LDAP configuration
roles/openstack/keystone/ssl - Perform tests of keystone SSL configuration

```

_Context Aware Operations_

As we all use shells for most of our work, this kit is designed to make that easy.  It allows
the user to set a context easily and then make all further commands relative to that context.
For Metacloud, the easiest grouping is availability zone or AZ. Once this is set, you can
then run ansible commands and playbooks with minimal effort.

```bash
brmathes@BRMATHES-M-C0SW > az CLIENT1.pv
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ticket 5234
brmathes@BRMATHES-M-C0SW [CLIENT1.pv-5234] > ops
brmathes@BRMATHES-M-C0SW [CLIENT1.pv-5234] > ansible -m ping mcp
```

_Find the right information Quickly_

Use binary tools to access information in Zendesk and Zabbix quickly.

```
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > tickets
zstatu5928 - STAGE1 network issue                                         2017-03-27T16:03:45Z
5780 - change service URLs in STAGE1                                2017-03-23T18:24:13Z
5407 - Two availability zones named "stage1" and one named "enic"   2017-03-21T23:03:00Z
5435 - Testing Ticket, please ignore                                2017-03-23T22:02:36Z
5723 - Multi AZ - Dashboard cert renewal before March 11th 2017     2017-03-25T03:02:33Z
4718 - Cut existing clients over to SV2                             2017-03-24T21:40:45Z
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > zmaint
   1. TRIAL14.mc (TRIAL14.mc - All Hosts)
     Thu Mar 23 09:08:00 2017 -> Thu Mar 30 09:08:00 2017 (1 week)

   2. CLIENT3.pv  (CLIENT3.pv - All Hosts)
     Sun Mar 26 20:22:00 2017 -> Mon Mar 27 20:22:00 2017 (1 day)

   3. CLIENT2.pv  (CLIENT2.pv - All Hosts)
     Tue Mar 21 07:27:00 2017 -> Tue Mar 28 07:27:00 2017 (1 week)
```

_Standardized Processes_

Using Ansible, many roles and modules have been developed to assist and automate processes we
generally perform in operations.  Using standardized approaches to issue resolution and
management will lower risk and raise certainty.

```
brmathes@BRMATHES-M-C0SW [CLIENT1.pv-5435] > ansible-playbook playbooks/procedures/mhv/maintenance_prepare.yml -e host=mhv7.CLIENT1.pv.metacloud.in

brmathes@BRMATHES-M-C0SW [CLIENT1.pv-5435] > ansible-playbook playbooks/procedures/mhv/maintenance_return.yml -e host=mhv7.CLIENT1.pv.metacloud.in
```

_Password Management_

This is being developed in two phases.
- Initially we'll use the pass(1) program to store all passwords we need to use.  It uses GPG to encrypt passwords and store them locally in ~/.password-store
- OPS is working on using a centralized password manager to store AZ passwords easily. Even in this case, we'll likely need local per-user passwords for zabbix, zendesk and others that will be stored locally.

_Simple Building Blocks_
All the written roles are made to be simple to execute.  They're intentionally written to keep role-variables to a minimum so when reviewing playbooks.. the potential user can understand exactly what operations will be taking place and have a quick reference to build their own playbooks.

Examples

```yaml
  - { role: metacloud/ticket/comment, comment: "placing {{host}} into maintenance." }
  - { role: metacloud/maintenance/host, state: present }
  - { role: openstack/hypervisor/nova_compute, host: "{{host}}", state: disable }
  - { role: openstack/hypervisor/evacuate, host: "{{host}}" }
  - { role: openstack/hypervisor/neutron_agent, host: "{{host}}", state: disable,
       when: neutron_common_mode is defined }
  - { role: metapod/ceph/noout, state: set,
       when: cinder_ceph or nova_ceph }
```
