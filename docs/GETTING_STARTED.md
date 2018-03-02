# Getting Started 

So you've installed ansible-ops .. and thinking .. what now?  Lets get started with some simple tools that 
should make it easy to see the power of the system.

- [Context](#context)
- [Tickets](#tickets)
- [Shell Commands](#shell-commands)
- [Playbooks](#playbooks)

## Context 

Setting the context for your shell is really easy, you just need to know the AZ name:
```
brmathes@BRMATHES-M-C0SW > az CLIENT1.pv
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] >
```
This does several things for you:
- finds the AZ in the inventory
- sets your prompt to remind you where you are.
- setup ansible environment variables
   - ANSIBLE_HOME - sets to the home location for ansible-ops
   - ANSIBLE_INVENTORY - sets the location of the inventory for the AZ
   - ANSIBLE_VAULT_PASSWORD_FILE - set to a script which will call pass(1) to return a stored password
- enables the virtualenv for ansible-ops
- sets some quick aliases for you
   - ops - goto ansible-ops
   - inv - goto the group vars dir for this AZ
   - systems - goto ansible-systems and set that venv
   - ap - ansible-playbook
   

We can now get to a host in the AZ just my typing the simple hostname( even for sprint! ):
```
brmathes@BRMATHES-M-C0SW > az CLIENT1.pv
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > mcp1
Last login: Thu Feb  9 22:40:27 2017 from 208.90.61.135

System is UCSB-B200-M3
ubuntu 12.04
Last spine: 20:09:41 08-Feb-2017 release: 10226 version: 2.1.4
n21075lp@mcp1.CLIENT1.pv:~$
```

Using these shortcuts we can also run a quick command when needed
```
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > mcp1 uptime
 15:29:46 up 5 days, 19:14,  0 users,  load average: 0.23, 0.40, 0.47
```
   
Using the power of ansible  can do some quick things across hostgroups defined in the AZ:
```
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ansible -m shell -a 'uname -a' mcp
mcp1.CLIENT1.pv.hosted.in | success | rc=0 >>
Linux mcp1.CLIENT1.pv.hosted.in 3.10.0-327.el7.x86_64 #1 SMP 

mcp2.CLIENT1.pv.hosted.in | success | rc=0 >>
Linux mcp2.CLIENT1.pv.hosted.in 3.10.0-327.el7.x86_64 #1 SMP 

mcp3.CLIENT1.pv.hosted.in | success | rc=0 >>
Linux mcp3.CLIENT1.pv.hosted.in 3.10.0-327.el7.x86_64 #1 SMP 
```

You can find tickets for the current AZ using the tickets tool. It has a -v if you want more detail
```
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > tickets
4193 - CLIENT1 openstack 1.7.4 upgrade                    2017-02-06T13:55:41Z
4210 - CLIENT Port for DNS                                2017-02-10T19:01:57Z
4324 - CCR CLIENT for DNS slave and 1.7.3                 2017-02-09T19:04:59Z
4719 - CLIENT Network Drop                                2016-11-25T08:00:49Z
5411 - CLIENT1 Capacity Reports Failing                   2017-01-24T20:49:04Z
5521 - CLIENT1 migration to new zabbix                    2017-02-08T23:08:09Z
5536 - CLIENT1.pv OpenStack and OpenVPN updates           2017-02-14T17:48:38Z
```

## Tickets
You can add (or remove) a ticket to your context so that as you're working, updates are 
automagically sent to the ticket (when it makes sense) 

```
brmathes@BRMATHES-M-C0SW > az CLIENT1.pv
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ticket 1234
brmathes@BRMATHES-M-C0SW [CLIENT1.pv-1234] > az
brmathes@BRMATHES-M-C0SW [-1234] > ticket
brmathes@BRMATHES-M-C0SW >
```

Once you've done this .. and you wanna see what's going on with this ticket.. you can use the 
'tlog' tool.  
```
brmathes@BRMATHES-M-C0SW [CLIENT1.pv-4210] > tlog
========= Ticket: #4210
       AZ: CLIENT1                 Priority: priority_standard
Requestor: CLIENT Client Communications           Created: 2016-08-12T14:56:06Z
 Assigned: Branson Matheson           Updated: 2017-02-10T19:01:57Z
  Subject: CLIENT Port for DNS

| Hello Client,
|
|  We're transitioning to using a new DNS configuration, and thus we need to
....


-- Comment by Branson Matheson on 2017-02-03T18:24:00Z (Private)
| Put CLIENT1 into maintenance

-- Comment by Branson Matheson on 2017-02-02T16:59:36Z
| Start time is actually Friday 02/03/17 at 1900UTC ( 11AM PST ).
|
| Branson Matheson
| Cloud Systems Technical Lead

```

## Shell Commands
As noted above.. we can use the ansible tool as a distribitued shell to accomplish common tasks.  
For example:
```
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ansible -m shell -a "uptime" mcp
mcp3.CLIENT1.pv.hosted.in | success | rc=0 >>
 19:16:05 up 174 days,  8:54,  0 users,  load average: 3.28, 2.85, 2.88

mcp2.CLIENT1.pv.hosted.in | success | rc=0 >>
 19:16:05 up 104 days, 7 min,  0 users,  load average: 1.70, 1.79, 1.63

mcp1.CLIENT1.pv.hosted.in | success | rc=0 >>
 19:16:05 up 200 days, 21:55,  0 users,  load average: 1.79, 1.73, 1.55
```

You can use any module you'd like in a similar manner for quick commands
```
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ansible -m ping CLIENT1 | grep success | sort
mcp1.CLIENT1.pv.hosted.in | success >> {
mcp2.CLIENT1.pv.hosted.in | success >> {
mcp3.CLIENT1.pv.hosted.in | success >> {
mhv1.CLIENT1.pv.hosted.in | success >> {
mhv2.CLIENT1.pv.hosted.in | success >> {
mhv3.CLIENT1.pv.hosted.in | success >> {
mhv4.CLIENT1.pv.hosted.in | success >> {
mhv5.CLIENT1.pv.hosted.in | success >> {
mhv6.CLIENT1.pv.hosted.in | success >> {
mhv7.CLIENT1.pv.hosted.in | success >> {
mhv8.CLIENT1.pv.hosted.in | success >> {
```

## Playbooks
So there are a growing plethora of [playbooks](../playbooks/README.md) that can be used quickly 
to evaluate or work on an AZ. 

Lets do a quick network test. First lets see what this play will do:
```
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ap playbooks/tests/network/all.yml --list-tasks

playbook: playbooks/tests/network/all.yml

  play #1 (Test DNS setup for proper operation):  TAGS: []
    check for resoluton of key hosts  TAGS: []
    check for port access via firewall  TAGS: []

  play #2 (Test VPN setup for proper operation):  TAGS: []
    check for resoluton of key hosts  TAGS: []
    check for port access via firewall  TAGS: []

  play #3 (Test Stunnel setup for proper operation):  TAGS: []
    check for resoluton of key hosts  TAGS: []
    check for port access via firewall  TAGS: []
```

Now lets see who this would applies to:
```
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ap playbooks/tests/network/all.yml --list-hosts

playbook: playbooks/tests/network/all.yml

  play #1 (Test DNS setup for proper operation): host count=3
    mcp1.CLIENT1.pv.hosted.in
    mcp2.CLIENT1.pv.hosted.in
    mcp3.CLIENT1.pv.hosted.in

  play #2 (Test VPN setup for proper operation): host count=3
    mcp1.CLIENT1.pv.hosted.in
    mcp2.CLIENT1.pv.hosted.in
    mcp3.CLIENT1.pv.hosted.in

  play #3 (Test Stunnel setup for proper operation): host count=3
    mcp1.CLIENT1.pv.hosted.in
    mcp2.CLIENT1.pv.hosted.in
    mcp3.CLIENT1.pv.hosted.in
```

We can see this test is scoped to all the mcps.  Since it's not a destructive operation, lets run it
against 1 of the MCP's

```
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ap playbooks/tests/network/all.yml -l mcp1.CLIENT1.pv.hosted.in

PLAY [Test DNS setup for proper operation] ************************************

TASK: [metapod/vpn/validation | check for resoluton of key hosts] *************
ok: [mcp1.CLIENT1.pv.hosted.in]

TASK: [metapod/vpn/validation | check for port access via firewall] ***********
ok: [mcp1.CLIENT1.pv.hosted.in]

PLAY [Test VPN setup for proper operation] ************************************

TASK: [metapod/vpn/validation | check for resoluton of key hosts] *************
ok: [mcp1.CLIENT1.pv.hosted.in]

TASK: [metapod/vpn/validation | check for port access via firewall] ***********
ok: [mcp1.CLIENT1.pv.hosted.in]

PLAY [Test Stunnel setup for proper operation] ********************************

TASK: [metapod/stunnel/validation | check for resoluton of key hosts] *********
ok: [mcp1.CLIENT1.pv.hosted.in] => (item={'connect_port': 8443, 'connect_host': 'stunnel1.mon1.in1.hosted.com', 'name': 'zabbix-prod', 'accept': u'127.0.0.1:{# zabbix_connect_port #}', 'client': 'yes'})

TASK: [metapod/stunnel/validation | check for port access via firewall] *******
ok: [mcp1.CLIENT1.pv.hosted.in] => (item={'connect_port': 8443, 'connect_host': 'stunnel1.mon1.in1.hosted.com', 'name': 'zabbix-prod', 'accept': u'127.0.0.1:{# zabbix_connect_port #}', 'client': 'yes'})

PLAY RECAP ********************************************************************
mcp1.CLIENT1.pv.hosted.in : ok=6    changed=0    unreachable=0    failed=0
```

This test playbook is pretty simple.. and shows how we can aggregate playbooks effectively:
```
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > cat playbooks/tests/network/all.yml
- include: dns.yml
- include: vpn.yml
- include: stunnel.yml
```

This means we can also call them individually when needed:

```
brmathes@BRMATHES-M-C0SW [CLIENT1.pv] > ap playbooks/tests/network/dns.yml

PLAY [Test DNS setup for proper operation] ************************************

TASK: [metapod/vpn/validation | check for resoluton of key hosts] *************
ok: [mcp3.CLIENT1.pv.hosted.in]
ok: [mcp1.CLIENT1.pv.hosted.in]
ok: [mcp2.CLIENT1.pv.hosted.in]

TASK: [metapod/vpn/validation | check for port access via firewall] ***********
ok: [mcp2.CLIENT1.pv.hosted.in]
ok: [mcp3.CLIENT1.pv.hosted.in]
ok: [mcp1.CLIENT1.pv.hosted.in]
```



