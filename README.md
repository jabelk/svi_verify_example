# Learn by Doing: Service Health Verification & Importing Packages

NSO has a ton of features. This repository is the first of a series which will show a simple use case, along with a feature. The purpose is dual:

1. See NSO applied to a variety of configuration situations and see how versatile it is..
1. Learn something new and have an example to follow

This example is a service package deploying:

**SVI Configuration**

and the features I am showcasing are:

**Service Health Verifications**
**Importing Packages**

This example assumes a working knowledge of services and NSO. 

## Installation

[Reserve the NSO Reservable Sandbox](https://devnetsandbox.cisco.com/RM/Diagram/Index/43964e62-a13c-4929-bde7-a2f68ad6b27c?diagramType=Topology)

If you need to revisit some of the NSO Basics, you can [start here](https://developer.cisco.com/learning/lab/learn-nso-the-easy-way/step/1). 

Use some type of file transfer program or [VS Code has remote SSH](https://code.visualstudio.com/docs/remote/ssh) (drag and drop the package into the packages directory)

This project requires **two** repos. The first is this one **(svi_verify_example)**. The second is the **[selftest package](https://github.com/NSO-developer/selftest)**.

Both packages should go here:

`/var/opt/ncs/packages/`

If you are reproducing this elsewhere, here are the version details:
```
using NSO 5.3.0.1
cisco-asa-cli-6.8
cisco-iosxr-cli-7.20
on 
Nexus9000 9.2(3)
ASAv 9.12(2)
```

After both packages are on the configured NSO system install server (10.10.20.49) at the packages folder, make sure the yang is compiled reload packages and configure a sample service instance and sample verification.

### Configure service

First, you will need to compile the yang and reload the packages, then create a service instance of the SVI package:

```
** transfer selftest AND svi_verify_example package over **
cd /var/opt/ncs/packages/svi_verify_example/src
make
cd /var/opt/ncs/packages/selftest/src
make
ncs_cli
packages reload
conf
svi_verify_example first-instance vlan-id 1337 vlan-name SVI-DEMO switches first-sw switch-device dist-sw01
exit
switches second-sw switch-device dist-sw02
exit
firewalls first-fw firewall-device edge-firewall01
exit
commit dry-run outformat native
commit
end
```

### Configure and run selftest

After the service instance is deployed and committed, you can populate and run selftest. This will be explained after the output section. 

```
show running-config svi_verify_example first-instance | display xpath
conf
selftest svi-tests service /svi_verify_example[name='first-instance']
commands ping-test command ping arguments 172.16.107.2 failstring "No route to host" devices dist-sw01
exit
commands sh-ip-int-br command show arguments "ip interface brief" devices dist-sw01
exit
commands test-svi-fail command ping arguments 172.16.110.2 devices dist-sw01 failstring "No route to host"
top
commit
selftest svi-tests execute
end
show svi_verify_example selftest-result
```

### Sample Output for SVI Service

here is some sample output:

```
[developer@nso ]$ cd /var/opt/ncs/packages/svi_verify_example/src
[developer@nso src]$ make
/opt/ncs/current/bin/ncsc  `ls svi_verify_example-ann.yang  > /dev/null 2>&1 && echo "-a svi_verify_example-ann.yang"` \
             --yangpath ../../selftest/src/yang -c -o ../load-dir/svi_verify_example.fxs yang/svi_verify_example.yang
[developer@nso src]$ cd /var/opt/ncs/packages/selftest/src/
[developer@nso src]$ make
/opt/ncs/current/bin/ncsc  `ls svi_verify_example-ann.yang  > /dev/null 2>&1 && echo "-a svi_verify_example-ann.yang"` \
             --yangpath ../../selftest/src/yang -c -o ../load-dir/svi_verify_example.fxs yang/svi_verify_example.yang
[developer@nso src]$ ncs_cli

developer connected from 192.168.254.11 using ssh on nso
developer@ncs# packages reload

>>> System upgrade is starting.
>>> Sessions in configure mode must exit to operational mode.
>>> No configuration changes can be performed until upgrade has completed.
>>> System upgrade has completed successfully.
reload-result {
    package cisco-asa-cli-6.8
    result true
}
reload-result {
    package cisco-ios-cli-6.44
    result true
}
reload-result {
    package cisco-iosxr-cli-7.20
    result true
}
reload-result {
    package cisco-nx-cli-5.15
    result true
}
reload-result {
    package resource-manager
    result true
}
reload-result {
    package selftest
    result true
}
reload-result {
    package svi_verify_example
    result true
}
developer@ncs#
System message at 2020-11-17 14:38:28...
    Subsystem stopped: ncs-dp-39-cisco-nx-cli-5.15:NexusDp
developer@ncs#
System message at 2020-11-17 14:38:28...
    Subsystem stopped: ncs-dp-38-cisco-ios-cli-6.44:IOSDp
developer@ncs#
System message at 2020-11-17 14:38:28...
    Subsystem stopped: ncs-dp-40-resource-manager:AddressallocationIPvalidation
developer@ncs#
System message at 2020-11-17 14:38:28...
    Subsystem stopped: ncs-dp-37-cisco-asa-cli-6.8:ASADp
developer@ncs#
System message at 2020-11-17 14:38:28...
    Subsystem started: ncs-dp-41-cisco-asa-cli-6.8:ASADp
developer@ncs#
System message at 2020-11-17 14:38:28...
    Subsystem started: ncs-dp-42-cisco-ios-cli-6.44:IOSDp
developer@ncs#
System message at 2020-11-17 14:38:28...
    Subsystem started: ncs-dp-43-cisco-nx-cli-5.15:NexusDp
developer@ncs#
System message at 2020-11-17 14:38:28...
    Subsystem started: ncs-dp-44-resource-manager:AddressallocationIPvalidation
developer@ncs#
developer@ncs#
developer@ncs# conf
Entering configuration mode terminal
developer@ncs(config)# svi_verify_example first-instance vlan-id 1337 vlan-name SVI-DEMO switches first-sw switch-device dist-sw01
developer@ncs(config-switches-first-sw)# exit
developer@ncs(config-svi_verify_example-first-instance)# switches second-sw switch-device dist-sw02
developer@ncs(config-switches-second-sw)# exit
developer@ncs(config-svi_verify_example-first-instance)# firewalls first-fw firewall-device edge-firewall01
developer@ncs(config-firewalls-first-fw)# exit
developer@ncs(config-svi_verify_example-first-instance)# commit dry-run outformat native
native {
    device {
        name dist-sw01
        data vlan 1337
              name SVI-DEMO
             !
             interface Vlan1337
              no shutdown
              description SVI-DEMO
              ip address 172.16.107.2/24
              no ip redirects
              ip router ospf 1 area 0.0.0.0
              no ipv6 redirects
              hsrp 10 ipv4
               ip 172.16.107.1
              exit
             exit
    }
    device {
        name dist-sw02
        data vlan 1337
              name SVI-DEMO
             !
             interface Vlan1337
              no shutdown
              description SVI-DEMO
              ip address 172.16.107.2/24
              no ip redirects
              ip router ospf 1 area 0.0.0.0
              no ipv6 redirects
              hsrp 10 ipv4
               ip 172.16.107.1
              exit
             exit
    }
    device {
        name edge-firewall01
        data object network INSIDE-SVI-DEMO
              subnet 172.16.107.0 255.255.255.0
             exit
             object-group network INSIDE-NETWORKS
              network-object object INSIDE-SVI-DEMO
             !
    }
}
developer@ncs(config-svi_verify_example-first-instance)# commit
Commit complete.
developer@ncs(config-svi_verify_example-first-instance)# end
```

## Verifying with the custom Selftest package

```
developer@ncs# show running-config svi_verify_example first-instance | display xpath
/svi_verify_example[name='first-instance']/switches[id='first-sw']/switch-device dist-sw01
/svi_verify_example[name='first-instance']/switches[id='second-sw']/switch-device dist-sw02
/svi_verify_example[name='first-instance']/firewalls[id='first-fw']/firewall-device edge-firewall01
/svi_verify_example[name='first-instance']/vlan-id 1337
/svi_verify_example[name='first-instance']/vlan-name SVI-DEMO
developer@ncs# conf
Entering configuration mode terminal
developer@ncs(config)# selftest svi-tests ?
Possible completions:
  commands
  execute
  service    Path to the service where the selftest result will be stored
  <cr>
developer@ncs(config)# selftest svi-tests service /svi_verify_example[name='first-instance']
developer@ncs(config-selftest-svi-tests)# commands ?
Possible completions:
  <name:string>
developer@ncs(config-selftest-svi-tests)# commands ping-test ?
Possible completions:
  arguments
  command
  devices
  failstring   regular expression for a fail result
  <cr>
developer@ncs(config-selftest-svi-tests)# commands ping-test command ping arguments 172.16.107.2 failstring "No route to host" devices dist-sw01
developer@ncs(config-commands-ping-test)# exit
developer@ncs(config-selftest-svi-tests)# commands sh-ip-int-br command show arguments "ip interface brief" devices dist-sw01
developer@ncs(config-commands-sh-ip-int-br)# exit
developer@ncs(config-selftest-svi-tests)# commands test-svi-fail command ping arguments 172.16.110.2 devices dist-sw01 failstring "No route to host"
developer@ncs(config-commands-test-svi-fail)# exit
developer@ncs(config-selftest-svi-tests)# top
developer@ncs(config)# commit
Commit complete.
developer@ncs(config)# selftest
Possible completions:
  <name:string>  svi-tests 
developer@ncs(config)# selftest svi-tests execute
result
output from ping
device: dist-sw01

PING 172.16.107.2 (172.16.107.2): 56 data bytes
64 bytes from 172.16.107.2: icmp_seq=0 ttl=255 time=1.752 ms
64 bytes from 172.16.107.2: icmp_seq=1 ttl=255 time=0.594 ms
64 bytes from 172.16.107.2: icmp_seq=2 ttl=255 time=1.217 ms
64 bytes from 172.16.107.2: icmp_seq=3 ttl=255 time=1.34 ms
64 bytes from 172.16.107.2: icmp_seq=4 ttl=255 time=1.173 ms

--- 172.16.107.2 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 0.594/1.215/1.752 ms

output from show
device: dist-sw01


IP Interface Status for VRF "default"(1)
Interface            IP Address      Interface Status
Vlan101              172.16.101.2    protocol-up/link-up/admin-up
Vlan102              172.16.102.2    protocol-up/link-up/admin-up
Vlan103              172.16.103.2    protocol-up/link-up/admin-up
Vlan104              172.16.104.2    protocol-up/link-up/admin-up
Vlan105              172.16.105.2    protocol-up/link-up/admin-up
Vlan1337             172.16.107.2    protocol-up/link-up/admin-up
Eth1/3               172.16.252.1    protocol-up/link-up/admin-up
Eth1/4               172.16.252.5    protocol-up/link-up/admin-up

output from ping
device: dist-sw01

PING 172.16.110.2 (172.16.110.2): 56 data bytes
ping: sendto 172.16.110.2 64 chars, No route to host
Request 0 timed out
ping: sendto 172.16.110.2 64 chars, No route to host
Request 1 timed out
ping: sendto 172.16.110.2 64 chars, No route to host
Request 2 timed out
ping: sendto 172.16.110.2 64 chars, No route to host
Request 3 timed out
ping: sendto 172.16.110.2 64 chars, No route to host
Request 4 timed out

--- 172.16.110.2 ping statistics ---
5 packets transmitted, 0 packets received, 100.00% packet loss

developer@ncs(config)# exit
developer@ncs# show svi_verify_example selftest-result ?
Description: last results for the service selftest action
Possible completions:
  command  |  <cr>
developer@ncs# show svi_verify_example selftest-result
NAME            NAME           RESULT  TIME
------------------------------------------------------------
first-instance  ping-test      OK      2020-11-17 15:19:52
                sh-ip-int-br   OK      2020-11-17 15:19:52
                sh-ip-test     OK      2020-11-17 15:14:23
                test-svi-fail  FAIL    2020-11-17 15:20:05

developer@ncs#
```

## Service Design Considerations

This Learn by Doing example is not so much about the service design, but the integration of verification. As such, the Yang and Templates are relatively simple, reusing concepts from previous Learn by Doing repos.

> I originally created this package with the expectation I would write my own health checks, so there is skeleton Python code in the `python` directory. It was not used, since I ended up using the `selftest` package. 

The configuration template loops over each device in the switch and firewall device list, with some basic inputs for VLAN ID and VLAN Name to be consistent between the firewalls and switches. The IP Addresses are static for the sake of development time, and keeping the example simple. They should be replaced with more sophisticated inputs if that is desired:

```xml
<config-template xmlns="http://tail-f.com/ns/config/1.0">
  <devices xmlns="http://tail-f.com/ns/ncs">
        <?foreach {/switches}?>
               <device>
                 <name>{switch-device}</name>
                 <config>
                   <vlan xmlns="http://tail-f.com/ned/cisco-nx">
                     <vlan-list>
                       <id>{/vlan-id}</id>
                       <name>{/vlan-name}</name>
                     </vlan-list>
                   </vlan>
                   <interface xmlns="http://tail-f.com/ned/cisco-nx">
                     <Vlan>
                       <name>{/vlan-id}</name>
                       <description>{/vlan-name}</description>
                       <ip>
                         <address>
                           <ipaddr>172.16.107.2/24</ipaddr>
                         </address>
                         <redirects>false</redirects>
                         <router>
                           <ospf>
                             <name>1</name>
                             <area>0.0.0.0</area>
                           </ospf>
                         </router>
                       </ip>
                       <ipv6>
                         <redirects>false</redirects>
                       </ipv6>
                       <hsrp>
                         <hsrp-list>
                           <id>10</id>
                           <addr_type>ipv4</addr_type>
                           <ip>
                             <address>172.16.107.1</address>
                           </ip>
                         </hsrp-list>
                       </hsrp>
                     </Vlan>
                   </interface>
                 </config>
               </device>
              <?end?> <!-- end foreach {/endpoint} -->
             </devices>


<devices xmlns="http://tail-f.com/ns/ncs">
        <?foreach {/firewalls}?>
               <device>
                 <name>{firewall-device}</name>
                 <config>
                   <object xmlns="http://cisco.com/ned/asa">
                     <network>
                       <name>INSIDE-{/vlan-name}</name>
                       <subnet>
                         <address>172.16.107.0</address>
                         <mask>255.255.255.0</mask>
                       </subnet>
                     </network>
                   </object>
                   <object-group xmlns="http://cisco.com/ned/asa">
                     <network>
                       <id>INSIDE-NETWORKS</id>
                       <network-object>
                         <id>object INSIDE-{/vlan-name}</id>
                       </network-object>
                     </network>
                   </object-group>
                 </config>
               </device>
              <?end?> <!-- end foreach {/endpoint} -->
             </devices>
</config-template>

```

The data model imports `selftest`, which is the custom verification package we downloaded and compiled. It also includes `uses selftest:selftest-result;` in the service so that we can store the operational values of the output in our service. 

```javascript
module svi_verify_example {

  namespace "http://example.com/svi_verify_example";
  prefix svi_verify_example;

  import ietf-inet-types {
    prefix inet;
  }
  import tailf-common {
    prefix tailf;
  }
  import tailf-ncs {
    prefix ncs;
  }
  // assumes makefile has been updated
  import selftest {
    prefix selftest;
  }


  description
    "Bla bla...";

  revision 2016-01-01 {
    description
      "Initial revision.";
  }

  list svi_verify_example {
    description "This is an RFS skeleton service";

    key name;
    leaf name {
      tailf:info "Unique service id";
      tailf:cli-allow-range;
      type string;
    }
    // gives us linkage to selftest
    uses selftest:selftest-result;
    uses ncs:service-data;
    ncs:servicepoint svi_verify_example-servicepoint;
  
   list switches {
        key "id";
        leaf id{
          tailf:info "Switches identifier";
          type string;
        }
        leaf switch-device {
          mandatory true;
          type leafref {
            path "/ncs:devices/ncs:device/ncs:name";
          }
        }
      }
   list firewalls {
        key "id";
        leaf id{
          tailf:info "firewalls identifier";
          type string;
        }
        leaf firewall-device {
          mandatory true;
          type leafref {
            path "/ncs:devices/ncs:device/ncs:name";
          }
        }
      }
      
      leaf vlan-id {
        type uint16;
        mandatory true;
      }
      leaf vlan-name{
        type string;
        mandatory true;
      }

  }
}
```

### Changes to Makefile

In order for the SVI Service to include the Yang files of the `selftest` package, I changed the Makefile in the SVI service to include this statement
```
YANGPATH += ../../selftest/src/yang
```
This tells NSO to pull in the data models from the other package. Since we then imported it, and used it, we now can use this package's functionality in our package. 


## Service Verification

There are multiple ways to verify a service health. You can include some Python actions, writing your own tests, or use some type of outside (not NSO) application to check on the health of the network. 

In this case, we are reusing the framework package written by Hakan, in order to save development time and also demonstrate reusing another package in our own service. 

The `selftest` package has a number of different features, but for our purposes, it is an easy to use wrapper around some Python code that allows us to define a list of verification operational commands to send, and some basic REGEX to see if there is a failure scenario in the output. 

The package uses the NSO `live-status` functionality to send commands to devices. I picked a few commands to send, showing successful and relevant tests, as well as one test that fails on purpose. The commands I chose were:


```
ping 172.16.107.2 (which should pass)
show ip interface brief (which should show our SVI)
ping 172.16.110.2 (which should fail)
```

The package checks to see if certain `failstring`s are present, for the ping test that was easy. For the `show ip interface brief`, we want to see that something is present for it succeed (the SVI being there is good), so it will always pass, but it gives us a record of the output to see the SVI is there. 

The pass and fail records are visible in our service because we have linked them together with the `uses` statement:

```
developer@ncs# show svi_verify_example selftest-result
NAME            NAME           RESULT  TIME
------------------------------------------------------------
first-instance  ping-test      OK      2020-11-17 15:19:52
                sh-ip-int-br   OK      2020-11-17 15:19:52
                sh-ip-test     OK      2020-11-17 15:14:23
                test-svi-fail  FAIL    2020-11-17 15:20:05

developer@ncs#
```

Note that the `test-svi-fail` test which was designed to fail, did indeed fail since the IP address `172.16.110.2` was not provisioned for this service, but the `172.16.107.2` one was and it passed. 