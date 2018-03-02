# Network Booting

When performing installations, Operations has consistent issues with network connectivity to/from the CIMC 
based hosts in an AZ.  This is due to distance, and other network limitations that aren't easily 
surmountable.  This impacts at least 2 types of installations:

* Firmware Updates - where we're trying to push updates to CIMC 
* MCP first boot - where we're trying to boot the first host 

To resolve this, and working with Network Engineering, a methodology was created for booting systems
via an HTTP server on a Nixes 9000 switch within the AZ.  While this seems like a logical way to 
provide the service, there were several complications:

* Network Routing - the "out of band" (OOB) networks are strictly controlled in both accessibility and 
routing.  Specifically:
   * Only inbound connections are allowed to the AZ on ports 22, 80 and 443
   * NO outbound connections are allowed back to MC services via OOB
   * There is no routing allowed between OOB and any other network in the AZ

* Switch TCP window size - the native configuration has a limited window size for access, which means
  moving data takes significantly longer.
  
# Design

To solve these issues, a design was created that utilized the N9k as an HTTP service in the local
subnet .. which could be setup and torn down quickly and efficiently

## Access
* The system is logged in as the native operations user, and then a shell is gained on the back end of
the switch using configured features.  
* An "mc_services" user is created on the host that allows remote connection via ansible and SCP.  Ansible modules cannot
be run on the switch as it's missing significant python tools.  Ansible can use the "raw" and "expect" modules
to manipulate the command line.
* the mc_service user is only accessible via the current operators SSH key. 
* an mc_services directory is created on the /bootflash partition to hold all the data/config for the service. 

## Copy Services
* An SSH service is created to run on the HTTPS port within the AZ. This service is restricted to only 
the mc_services user
* It' setup to run in the 'manageament' netns context so the anodizes is not limited

## HTTP Services
* an HTTP service is created to run on the HTTP port on the switch.  This service is restricted to read-only and 
services data directory is /bootflash/mc_service/nginx/html 
* It' setup to run in the 'manageament' net's context so the windowsize is not limited


## Operation
* The operator calls the enable tool .. which will start up all the services on the swtich
* The boot files are then copied to the switch via the mc_services users and the HTTPS port
* the systems are then booted using the http://{{ip}}/files as needed
* at completion the operator calls the clean up tool which removes all traces of the operation except logs. 