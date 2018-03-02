# Writing New Playbooks

## Requirements
To write new playbooks we have a few requirements:

1. Each playbook must be constrained in a section so that it is easy to identify scope and intent.
1. Each playbook must test for any required variables using assert.
1. High-risk playbooks ( such as those that could impact an entire cluster ) should have a challenge to make sure the operations person really wants to do it.
1. Check your new playbook into a branch and submit a PR for inclusion.  Drop a note on ansible-ops if you want. 

## Testing for variables
Ansbible has a nice construct for doing this using assert at the beginning of a playbook.

```yaml
  - name: test for runtime data
    hosts: mcp
    gather_facts: no
    tasks:
      - assert:
          that:
            - "'{{lookup('env','AZ')}}' != ''"
            - "'{{lookup('env','TICKET')}}' != ''"
            - "{{play_hosts|length}} == 1"
        tags: [check_vars]
```

Some useful assertions:

- Test for environment variable
```
           "'{{lookup('env','AZ')}}' != ''"
```

- Test for numbers of hosts
```
           "{{play_hosts|length}} == 1"
```


## Challenge Response
Some operations are more risky than others, and implmenting a challenge to a command can help reduce risk by adding a step for operations.
- use a vars_prompt requesting a piece of info relative to the operation
- set a fail task if the challenge doesn't match

This is accomplised easily using ansible thusly:

```yaml
  - name: test for runtime data
    hosts: mcp
    gather_facts: no
    vars_prompt:
      - name: "verify"
        prompt: "Type in the FQDN of the host {{inventory_hostname}} to reboot: "
        private: no
    tasks:
      - fail: msg="FQDN does not match hostname"
        when: "'{{verify}}' != '{{inventory_hostname}}'"
```
