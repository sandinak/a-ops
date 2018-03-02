# Requirements

This is the list of requirements defined for the ansible-ops system

## Shell
1. Users shall be able to define the AZ they're working on in each shell and tools will be use the data.
1. Context setting shall understand both Spine and Ansible inventory and work correctly and (mostly) transparently with each.
1. Context based commands run when no context is set will throw a useful error.

## Setup
1. Installation shall be simple and well documented for MacOS and Linux.
1. Setup of links will be simple and scripted.
1. Tools required shall be minimal ( ansible, git, pass ).

## Documentation
1. All playbooks, roles, library modules will be documented in markdown format adjacent the files.

## Playbooks
1. Playbooks will be documented in comments at the top of the file ( ansible doesn't have a feature for this yet .. darnit ).
1. Playbooks will test for required variables at the top of the playbook using assert.
1. Each playbook will update a relevant ticket with status/changes where appropriate and when defined.
1. Playbooks that will impact zabbix will set an appropriate maintenance so alerts are not generated.
1. Playbooks will send notifications of major changes to slack where appropriate.
