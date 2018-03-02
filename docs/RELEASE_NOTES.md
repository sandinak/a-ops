# Versions

v2.1 - Liberty Upgrade Updates
  - added playbooks to handle liberty updates
    - handles running ansible-systems playbooks using venv
    - handles repeated running using basic step management
    - each update playbook set checks for required variables.
    - playbook run output is put in a ticket dir
    - playbook runs can be tracked using 'trunlog' 
  - wrote tool to create mysql backups
    - wrote dependency that creates scratch directory and purge cronjob
  - wrote tools to manage ansible-inventory
    - branch a current installation
    - update non-encrypted variables in place
  - wrote tools to manage ansible-systems
    - branch to a correct version and reset venv to match
    - run a playbook in the venv 
  - created step tracking
    - created role for tracking runs by UUID
    - created step module for tracking steps 
  - added tools for sprint 
    - flushing SSSD when it goes wonky.
  - added some testing tools
    - playbook to dump all ansible vars to a tmpfile
    - 
    

v2.0 - Documentation and ticket tools
  - added adoc and refactored documentation 
    - handles filters, roles, playbooks, binaries and modules
    - automagically rebuilds documentation when adding tools and roles
    - handles keywords cleanly
    - stores all general information for a directory tree(s) in adoc.yml in root
    - uses ansible native tools to process documentation strings
  - added new playbooks for spine
    - pushing users
    - managing certs
  - added tools for ticket updating
    - tentry to add a comemnt to a ticket
    - tattach to add a file to a ticket
  
    

v1.2 - UCS tools
  - integrated imcsdk and ansible-imc to create tools
    - cisco_imc_ipmi - manage ipmi
    - cisco_imc_sol - manage SOL
    - cisco_imc_server - manage power and locator led
  - wrote tools for setting up and grabbing consoles
    - playbooks/operations/cimc/enable_sol_console
    - playbooks/operations/cimc/get_sol_consoles
  - wrote 'console' helper scripts for getting the console easily
  - wrote 'secrets' function to get to the passwords more easily
  - rewrote spine apply to handle multiple hosts correctly 
  - extended documentation
  

v1.1 - Install Fixes (02/15/2017)
  - updated installer to use venv
    - handles requirements
  - cleaned up opsrc to not pollute env with unneeded vars
  - added new C/L tools
    - tickets - tickets for current AZ
    - tlog - ticket log for defined ticket
    - zstatus - any current issues with current AZ
    - zmaint - list of all maintenances
  - added some basic tests
    - playbooks/tests/network 
  - refined spine tools to work cleanly for mhv and mcp
  - expanded documentation
    

V1.0 - Initial Release (02/03/2017)
  - created documentation
  - created install process
  - Tools
    - build_host_links
    - ops_install - check and validate ansible-ops install
    - zabbix-maintenance-report - return list of current maintenances
  - Playbooks
    - Installation support
      - ansible-ops - check installation for all tools and configuration
      - hostlinks.yml - expand links to include all hosts in current AZ
      - update.yml - update this repo
    - Quick operations
      - operations/az/mainteance_set
      - operations/az/mainteance_remove
      - operations/mcp/mainteance_set
      - operations/mcp/mainteance_remove
      - operations/mhv/mainteance_set
      - operations/mhv/mainteance_remove
    - Manage maintenance procedures
      - procedures/mcp/maintenance_prepare.yml
      - procedures/mcp/maintenance_return.yml
      - procedures/mhv/maintenance_prepare.yml
      - procedures/mhv/maintenance_return.yml
      - procedures/az/poweroff.yml
    - Tests
      - tests/keystone/ldap - verify LDAP and SSL config on an AZ

