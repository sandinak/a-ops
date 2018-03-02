# Usage

Using this tool kit is pretty simple

## Context Management
- when opening a new shell set your AZ.  You may be prompted for the ansible_vault_password which lives in PassPack and will be stored in pass(1) using GPG.
```bash
  brmathes@BRMATHES-M-C0SW > az CLIENT1.pv
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv] >
```
- working on a ticket, can set that so that changes will get automagically updated as private-comments.

```bash
  # set it
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ticket 8088
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv-8088] >

  # change tickets
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv-8088] > ticket 1234
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv-1234] >

  # unset ticket
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv-8088] > ticket 1234
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv] >
```

- changing to systems context

```bash
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > systems
  brmathes@BRMATHES-M-C0SW [S-CLIENT1.pv] >
```

- adding openstack context
```bash
  brmathes@BRMATHES-M-C0SW [S-CLIENT1.pv] > stack
  Using /Users/brmathes/Documents/openstack-environments/current
  brmathes@BRMATHES-M-C0SW [S-CLIENT1.pv-current] >
```

- to access the MCP or MHV .. can just use the shortcut. Works with Sprint too!

```bash
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > mcp1
  brmathes@mcp1.CLIENT1.pv ~$
```

- to access an associated CIMC for an MCP or MHV

```bash
  brmathes@BRMATHES-M-C0SW [ngena11.pv] > mcp1.oob
  Connecting to admin@mcp1.oob.ngena11.pv.metacloud.in ...
  admin@mcp1.oob.ngena11.pv.metacloud.in's password:
  mcp1.oob.ngena11#
```

- to run commands on groups of hosts:

```bash
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ops
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ansible -m shell -a uptime mcp
```

- to run a playbook from ops:

```bash
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ops
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ansible-playbook playbooks/az/set_maintenance.yml \
    -e minutes=90 -e ticket=5704 -e state=present
``

- to run a playbook from systems

```bash
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > systems
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ansible-playbook playbooks/openstack/metapod.yml
```

- to run a playbook from users
```bash
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ausers
brmathes@BRMATHES-M-C0SW [U-CLIENT1.pv] > ap playbooks/deploy_users.yml
```

## Running Procedures

- for instance .. setting a procedure to put an MHV in maintenance

```bash
  brmathes@BRMATHES-M-C0SW > az CLIENT1.pv
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ticket 8088
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv-8088] > ops
  brmathes@BRMATHES-M-C0SW [CLIENT1.pv-8088] > ansible-playbook playbooks/procedures/mhv/maintenance_prepare.yml -e host=mhv12.CLIENT1.pv.metacloud.in
```
