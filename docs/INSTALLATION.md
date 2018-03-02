# Installation

Installing this tool is easy, and it's designed to be transparent for day-to-day

- follow the instructions for the OS you're running to get the base requirements in place.
    - [OSX](INSTALL_OSX.md) - OSX
    - [Linux](INSTALL_linux.md) - Linux

- Checkout the code

```bash
  user@foo > cd ~/git ; git clone git@github.com:metacloud/ansible-ops.git
```


- Edit your dotfiles 
   - set env variables with the right info:
```bash
  #-- metacloud
  #
  # define these before you source this file, or if you
  # copy this file .. then just uncomment and update.
  #
  #-- where your git checkouts live
  export GIT_HOME=~/git
  
  #-- where spine lives
  export SPINE_DIR=${GIT_HOME}/spine 
  
  #-- where tickets should be logged
  export TICKETS_LOG_DIR=~/Documents/Metacloud/tickets
  
  #-- the ansible directories, used by further scripts.
  export OPS_HOME=${GIT_HOME}/ansible-ops
  export AI=${GIT_HOME}/ansible-inventory
  #
  #-- a place to put links to shortcut 'mcp' and 'mhv' based on AZ
  export HOST_LINKS=~/bin/mc_hosts
  export PATH=${HOST_LINKS}:${OPS_HOME}/bin:$PATH
  #
  #
  #-- a place to have local openstack binaries to use against AZs
  #   (optional) 
  export STACK_HOME=~/Documents/openstack-environments
  
  source ${OPS_HOME}/skel/opsrc
```

   - Set your prompt to be context aware
      - bash - someone wanna write this? .. I found a few replacements for precmd in bash.. but none that I liked.
      - liquidprompt - hack to support az/ticket (there might be a better way?)
      - zsh - works to set the titlebar and the regular prompt
      
   Some Notes:
   - The prompt is now cleanly defined as $MC_TXT .. which is colon delimited context info.  you can also set your prompt by hand. Here are the variables
      - CONTEXT : blank for aops, U for users, S for Systems
      - AZ : the name of the az ( eg .. nfv1.pv ) 
      - TICKET : the ticket number
      - STACK_ENV : the openstack env 
        
  _liquidprompt patch_


```bash
  --- /usr/local/share/liquidprompt 2016-06-25 01:09:32.000000000 -0700
  +++ liquidprompt  2017-07-25 17:48:51.000000000 -0700
  @@ -390,6 +390,8 @@
       LP_COLOR_IN_MULTIPLEXER=${LP_COLOR_IN_MULTIPLEXER:-$BOLD_BLUE}
       LP_COLOR_RUNTIME=${LP_COLOR_RUNTIME:-$YELLOW}
       LP_COLOR_VIRTUALENV=${LP_COLOR_VIRTUALENV:-$CYAN}
  +    LP_COLOR_AZ=${LP_COLOR_AZ:-$BOLD_GREEN}
  +    LP_COLOR_TICKET=${LP_COLOR_TICKET:-$RED}

       if [[ -z "${LP_COLORMAP-}" ]]; then
           LP_COLORMAP=(
  @@ -1834,6 +1836,10 @@
           # is set.
           PS1+="${LP_VCS}"

  +        # Add AZ and TICKET per ansible-ops
  +        [[ ! -z "${AZ}" ]] && PS1+=" ${LP_COLOR_AZ}${AZ}$NO_COL"
  +        [[ ! -z "${TICKET}" ]] && PS1+=" ${LP_COLOR_TICKET}${TICKET}$NO_COL"
  +
           # add return code and prompt mark
           PS1+="${LP_RUNTIME}${LP_ERR}${LP_MARK_PREFIX}${LP_COLOR_MARK}${LP_MARK}${LP_PS1_POSTFIX}"

```

 _zsh example_

```bash
  # prompt .. you can copy this where ever or just use it... sets titlebar
  #
  case $TERM in
    xterm*)
          precmd () {
        if [ -z "${MC_TXT}" ]; then
              export PS1="%{$fg[yellow]%}%B%n@${HOST}%B %(#.#.>)%b "
          echo -ne "\e]1; ($os) ${LOGNAME}@${FHOST}:${PWD} ${MC_TXT}\a"
        else
              export PS1="%{$fg[yellow]%}%B%n@${HOST}%{$fg[white]%} [${MC_TXT}]%{$reset_color%}%B %(#.#.>)%b "
          echo -ne "\e]1; ($os) ${LOGNAME}@${FHOST}:${PWD} [${MC_TXT}]\a"
        fi

      }
      # chpwd() { echo -ne "\e]1; ($os) ${LOGNAME}@${FHOST}:%~\a" }
          export PS1="%B%n@${HOST} %(#.#.>) %b "
      ;;
    *)
          export PS1="%B%n@${FHOST}:%~ %(#.#.>) [$AZ] %b "
      ;;
  esac
```

- start a new shell with the new variables and run the installation script

```bash
  user@foo > ops
  user@foo > bin/ops_install
```

ops_install will prompt you first for the 'Metacloud: Ansible Ops' password, available from Passpack. Then it will prompt you for the Zabbix Username and Zabbix Password. This is *your* Zabbix username and *your* Zabbix password. (Even though the username prompt says "Enter password for mc_services/zabbix/user_name," it's really looking for your username.)

```
 - initalizing pass
   What email address did you use for your GPG key:
jimbob@cisco.com
Password store initialized for jimbob@cisco.com
  - you will need the default password from passpack 'Metacloud: Ansible Ops'
    to install the default passwords for other services.
  - this will also pre-load the pass environment for you.
Enter password for mc_services/ansible-ops/ansible_vault_password:
Retype password for mc_services/ansible-ops/ansible_vault_password:
 - verifying user passwords
     mc_services/zabbix/user_name
Enter password for mc_services/zabbix/user_name:
Retype password for mc_services/zabbix/user_name:
     mc_services/zabbix/user_password
Enter password for mc_services/zabbix/user_password:
Retype password for mc_services/zabbix/user_password:
 - Running ansible to complete installation
 ```
