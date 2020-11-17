# Learn by Doing: Service Template Conditionals

NSO has a ton of features. This repository is the first of a series which will show a simple use case, along with a feature. The purpose is dual:

1. See NSO applied to a variety of configuration situations and see how versatile it is..
1. Learn something new and have an example to follow

This example is a service package deploying:

**NTP Configuration**

and the feature I am showcasing is:

**Service Template XML Conditionals**

This example assumes a working knowledge of services and NSO. 

## Installation

[Reserve the NSO Reservable Sandbox](https://devnetsandbox.cisco.com/RM/Diagram/Index/43964e62-a13c-4929-bde7-a2f68ad6b27c?diagramType=Topology)

If you need to revisit some of the NSO Basics, you can [start here](https://developer.cisco.com/learning/lab/learn-nso-the-easy-way/step/1). 

Use some type of file transfer program or [VS Code has remote SSH](https://code.visualstudio.com/docs/remote/ssh) (drag and drop the package into the packages directory)

This repo should go here:

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

After the package is on the configured NSO system install server (10.10.20.49) at the packages folder, make sure the yang is compiled reload packages and configure some sample service instances:

```
** transfer package over **
cd /var/opt/ncs/packages/svi_verify_example/src
make
ncs_cli
packages reload
conf
ntp_server_example first-instance endpoint 1st-dev ntp-device dist-rtr01
commit dry-run outformat native
region AMER
commit dry-run outformat native
exit
endpoint 2nd-dev ntp-device dist-rtr02 region APJC
exit
endpoint 3rd-dev ntp-device core-rtr01 region AMER
exit
endpoint 4th-dev ntp-device core-rtr02 region EMEA
exit
endpoint 5th-dev ntp-device dist-sw01 region AMER
exit
endpoint 6th-dev ntp-device dist-sw02 region APJC
exit
endpoint 7th-dev ntp-device edge-sw01 region EMEA
exit
endpoint 8th-dev ntp-device internet-rtr01 region AMER
commit dry-run outformat native
commit
```

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
developer@ncs(config)# svi_verify_example first-instance vlan-id 1337 vlan-name SVI-DEMO switches first-sw switch-device dist-sw01 ?
Possible completions:
  <cr>
developer@ncs(config)# svi_verify_example first-instance vlan-id 1337 vlan-name SVI-DEMO switches first-sw switch-device dist-sw01
developer@ncs(config-switches-first-sw)# exit
developer@ncs(config-svi_verify_example-first-instance)# switches second-sw switch-device dist-sw02
developer@ncs(config-switches-second-sw)# ?
Possible completions:
  switch-device
  ---
  commit          Commit current set of changes
  describe        Display transparent command information
  exit            Exit from current mode
  help            Provide help information
  no              Negate a command or set its defaults
  pwd             Display current mode path
  rload           Load configuration from an ASCII file relative to current location
  top             Exit to top level and optionally run command
developer@ncs(config-switches-second-sw)# exit
developer@ncs(config-svi_verify_example-first-instance)# firewalls first-fw firewall-device edge-firewall01
developer@ncs(config-firewalls-first-fw)# exit
developer@ncs(config-svi_verify_example-first-instance)# ?
Possible completions:
  check-sync           Check if device config is according to the service
  commit-queue
  deep-check-sync      Check if device config is according to the service
  firewalls
  get-modifications    Get the data this service created
  log
  re-deploy            Run/Dry-run the service logic again
  reactive-re-deploy   Reactive re-deploy of service logic
  switches
  touch                Mark the service as changed
  un-deploy            Undo the effects of the service
  vlan-id
  vlan-name
  ---
  commit               Commit current set of changes
  describe             Display transparent command information
  exit                 Exit from current mode
  help                 Provide help information
  no                   Negate a command or set its defaults
  pwd                  Display current mode path
  rload                Load configuration from an ASCII file relative to current location
  top                  Exit to top level and optionally run command
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
developer@ncs# show running-config svi_verify_example ?
Possible completions:
  first-instance   Unique service id
  |                Output modifiers
  <cr>
Possible match completions:
  firewalls  switches  vlan-id  vlan-name
developer@ncs# show running-config svi_verify_example fir
Possible completions:
  first-instance   Unique service id
Possible match completions:
  firewalls
developer@ncs# show running-config svi_verify_example first-instance | display xpath
/svi_verify_example[name='first-instance']/switches[id='first-sw']/switch-device dist-sw01
/svi_verify_example[name='first-instance']/switches[id='second-sw']/switch-device dist-sw02
/svi_verify_example[name='first-instance']/firewalls[id='first-fw']/firewall-device edge-firewall01
/svi_verify_example[name='first-instance']/vlan-id 1337
/svi_verify_example[name='first-instance']/vlan-name SVI-DEMO
developer@ncs# conf
Entering configuration mode terminal
developer@ncs(config)# selftest test-svi-ping service /svi_verify_example[name='first-instance']
developer@ncs(config-selftest-test-svi-ping)# ?
Possible completions:
  commands
  execute
  service    Path to the service where the selftest result will be stored
  ---
  commit     Commit current set of changes
  describe   Display transparent command information
  exit       Exit from current mode
  help       Provide help information
  no         Negate a command or set its defaults
  pwd        Display current mode path
  rload      Load configuration from an ASCII file relative to current location
  top        Exit to top level and optionally run command
developer@ncs(config-selftest-test-svi-ping)# comm
Possible completions:
  commands
  ---
  commit     Commit current set of changes
developer@ncs(config-selftest-test-svi-ping)# commands ?
Possible completions:
  <name:string>
developer@ncs(config-selftest-test-svi-ping)# commands ping-test ?
Possible completions:
  arguments
  command
  devices
  failstring   regular expression for a fail result
  <cr>
developer@ncs(config-selftest-test-svi-ping)# commands ping-test command ?
Possible completions:
  any  copy  nonconfig-actions  ping  show
developer@ncs(config-selftest-test-svi-ping)# commands ping-test command ping arguments 172.16.107.2 failstring "No route to host"
Value for 'devices' (list): dist-sw01
developer@ncs(config-commands-ping-test)# top
developer@ncs(config)# selftest test-svi-fail service /svi_verify_example[name='first-instance']
developer@ncs(config-selftest-test-svi-fail)# commands ping-test command ping arguments 172.16.110.2 failstring "No route to host"
Value for 'devices' (list): dist-sw01
developer@ncs(config-commands-ping-test)# top
developer@ncs(config)# selftest test-svi-sh-ip service /svi_verify_example[name='first-instance']
developer@ncs(config-selftest-test-svi-sh-ip)# commands sh-ip-test command show arguments "interface vlan" devices dist-sw01
developer@ncs(config-commands-sh-ip-test)# top
developer@ncs(config)# commit
Commit complete.
developer@ncs(config)# sefl
                       ^
% Invalid input detected at '^' marker.
developer@ncs(config)# selftest ?
This line doesn't have a valid range expression
Possible completions:
  <name:string>  test-svi-fail  test-svi-ping  test-svi-sh-ip
developer@ncs(config)# selftest test-svi-ping execute
result
output from ping
device: dist-sw01

PING 172.16.107.2 (172.16.107.2): 56 data bytes
64 bytes from 172.16.107.2: icmp_seq=0 ttl=255 time=1.578 ms
64 bytes from 172.16.107.2: icmp_seq=1 ttl=255 time=0.712 ms
64 bytes from 172.16.107.2: icmp_seq=2 ttl=255 time=1.763 ms
64 bytes from 172.16.107.2: icmp_seq=3 ttl=255 time=2.195 ms
64 bytes from 172.16.107.2: icmp_seq=4 ttl=255 time=0.996 ms

--- 172.16.107.2 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 0.712/1.448/2.195 ms

developer@ncs(config)# selftest test-svi-fail execute
result
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

developer@ncs(config)# selftest test-svi-
Possible completions:
  test-svi-fail  test-svi-ping  test-svi-sh-ip
developer@ncs(config)# selftest test-svi-sh-ip execute
result
output from show
device: dist-sw01

                              ^
Invalid command (incomplete interface) at '^' marker.

developer@ncs(config)# selftest test-svi-sh-ip ?
Possible completions:
  commands
  execute
  service    Path to the service where the selftest result will be stored
  <cr>
developer@ncs(config)# selftest test-svi-sh-ip commands ?
This line doesn't have a valid range expression
Possible completions:
  <name:string>  sh-ip-test
developer@ncs(config)# selftest test-svi-sh-ip commands sh-ip-test ?
Possible completions:
  arguments
  command
  devices
  failstring   regular expression for a fail result
  <cr>
developer@ncs(config)# selftest test-svi-sh-ip commands sh-ip-test ?
Possible completions:
  arguments
  command
  devices
  failstring   regular expression for a fail result
  <cr>
developer@ncs(config)# selftest test-svi-sh-ip commands sh-ip-test command ?
Possible completions:
  [show]  any  copy  nonconfig-actions  ping  show
developer@ncs(config)# selftest test-svi-sh-ip commands sh-ip-test command show ?
Possible completions:
  arguments
  devices
  failstring   regular expression for a fail result
  <cr>
developer@ncs(config)# selftest test-svi-sh-ip commands sh-ip-test command show arguments ?
Possible completions:
  <string>[interface vlan]
developer@ncs(config)# selftest test-svi-sh-ip commands sh-ip-test command show arguments "ip interface brief"
developer@ncs(config-commands-sh-ip-test)# top
developer@ncs(config)# commit
Commit complete.
developer@ncs(config)# selftest test-svi-sh-ip ?
Possible completions:
  commands
  execute
  service    Path to the service where the selftest result will be stored
  <cr>
developer@ncs(config)# selftest test-svi-sh-ip execute
result
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

developer@ncs(config)# end
developer@ncs# show running-config selftest
selftest test-svi-fail
 service /svi_verify_example[name='first-instance']
 commands ping-test
  devices    [ dist-sw01 ]
  command    ping
  arguments  172.16.110.2
  failstring "No route to host"
 !
!
selftest test-svi-ping
 service /svi_verify_example[name='first-instance']
 commands ping-test
  devices    [ dist-sw01 ]
  command    ping
  arguments  172.16.107.2
  failstring "No route to host"
 !
!
selftest test-svi-sh-ip
 service /svi_verify_example[name='first-instance']
 commands sh-ip-test
  devices   [ dist-sw01 ]
  command   show
  arguments "ip interface brief"
 !
!
developer@ncs# conf
Entering configuration mode terminal
developer@ncs(config)# selftest svi-tests ?
Possible completions:
  commands
  execute
  service    Path to the service where the selftest result will be stored
  <cr>
developer@ncs(config)# selftest svi-tests service /svi_verify_example[name='first-instance']
developer@ncs(config-selftest-svi-tests)# dev
                                          ^
% Invalid input detected at '^' marker.
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
developer@ncs(config-selftest-svi-tests)# commands ping-test command ping 172.16.107.2 no
                                                                          ^
% Invalid input detected at '^' marker.
developer@ncs(config-selftest-svi-tests)# commands ping-test command ping 172.16.107.2 ar
                                                                          ^
% Invalid input detected at '^' marker.
developer@ncs(config-selftest-svi-tests)# commands ping-test command ping 172.16.107.2 fa
                                                                          ^
% Invalid input detected at '^' marker.
developer@ncs(config-selftest-svi-tests)# commands ping-test command ping arguments 172.16.107.2 failstring "No route to host"
Value for 'devices' (list):
Error: user aborted
developer@ncs(config-selftest-svi-tests)# commands ping-test command ping arguments 172.16.107.2 failstring "No route to host" devices dist-sw01
developer@ncs(config-commands-ping-test)# exit
developer@ncs(config-selftest-svi-tests)# comm
Possible completions:
  commands
  ---
  commit     Commit current set of changes
developer@ncs(config-selftest-svi-tests)# commands sh-ip-int-br ?
Possible completions:
  arguments
  command
  devices
  failstring   regular expression for a fail result
  <cr>
developer@ncs(config-selftest-svi-tests)# commands sh-ip-int-br command show "ip interface brief" dev
                                                                             ^
% Invalid input detected at '^' marker.
developer@ncs(config-selftest-svi-tests)# commands sh-ip-int-br command show arguments ?
Possible completions:
  <string>
developer@ncs(config-selftest-svi-tests)# commands sh-ip-int-br command show arguments "ip interface brief" devices dist-sw01 ?
Possible completions:
  failstring   regular expression for a fail result
  <cr>
developer@ncs(config-selftest-svi-tests)# commands sh-ip-int-br command show arguments "ip interface brief" devices dist-sw01
developer@ncs(config-commands-sh-ip-int-br)# exit
developer@ncs(config-selftest-svi-tests)# comm
Possible completions:
  commands
  ---
  commit     Commit current set of changes
developer@ncs(config-selftest-svi-tests)# commands test-svi-fail ?
Possible completions:
  arguments
  command
  devices
  failstring   regular expression for a fail result
  <cr>
developer@ncs(config-selftest-svi-tests)# commands test-svi-fail command ?
Possible completions:
  any  copy  nonconfig-actions  ping  show
developer@ncs(config-selftest-svi-tests)# commands test-svi-fail command ping "172.16.110.2" dev
                                                                              ^
% Invalid input detected at '^' marker.
developer@ncs(config-selftest-svi-tests)# commands test-svi-fail command ping arguments 172.16.110.2
Possible completions:
  devices
  failstring   regular expression for a fail result
  <cr>
developer@ncs(config-selftest-svi-tests)# commands test-svi-fail command ping arguments 172.16.110.2 devices dist-sw01 failstring "No route to host" ?
Possible completions:
  <cr>
developer@ncs(config-selftest-svi-tests)# commands test-svi-fail command ping arguments 172.16.110.2 devices dist-sw01 failstring "No route to host"
developer@ncs(config-commands-test-svi-fail)# exit
developer@ncs(config-selftest-svi-tests)# top
developer@ncs(config)# commit
Commit complete.
developer@ncs(config)# selftest
Possible completions:
  <name:string>  svi-tests  test-svi-fail  test-svi-ping  test-svi-sh-ip
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
developer@ncs# show svi_verify_example te
                                       ^
% Invalid input detected at '^' marker.
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



## Service Template Conditional Statements

The conditional statement example can be found in the NSO Development guide PDF with the installation, in the  chapter 11 section called `Conditional Statements`. 

Basically conditional statements allow us to control which pieces of the configuration we want to exist or not, without using any Python/Java code. You will need to make some type of statement that evaluates to true or false. In our example we are checking to see which region the device is set to, and giving it that NTP server. 

The configuration we want to filter looks like this:
```
ntp server 10.20.30.40
```
Depending on which region our device sits, we want a different IP address for the server. 

In this service package we have the following data model:
```javascript
module ntp_server_example {
  namespace "http://com/example/ntp_server_example";
  prefix ntp_server_example;

  import ietf-inet-types {
    prefix inet;
  }
  import tailf-ncs {
    prefix ncs;
  }
  import tailf-common {
    prefix tailf;
  }

  list ntp_server_example {
    key name;

    uses ncs:service-data;
    ncs:servicepoint "ntp_server_example";

    leaf name {
      type string;
    }

   list endpoint {
        key "id";
        leaf id{
          tailf:info "Endpoint identifier";
          type string;
        }
        leaf ntp-device {
          mandatory true;
          type leafref {
            path "/ncs:devices/ncs:device/ncs:name";
          }
        }
        leaf region {
          type enumeration {
            enum "AMER";
            enum "APJC";
            enum "EMEA";
          }
        }


      }


    // // may replace this with other ways of refering to the devices.
    // leaf-list device {
    //   type leafref {
    //     path "/ncs:devices/ncs:device/ncs:name";
    //   }
    // }

    // // replace with your own stuff here
    // leaf dummy {
    //   type empty;
    // }
  }
}

```

I replaced the single device leafref with a list of `endpoints` which leafref back to the device list, but also give us an additional leaf of `region` to tag each one with a region for the conditional to check against. 

Since NTP servers are on pretty much every device, and don't change often, they are usually lumped together with other static config. For the sake of simplicity and instruction of concepts, they are the only configuration this service is using. 

### Foreach Device Looping

Even though this repo is focusing on conditionals, I took the opportunity to give an example of looping over a list of devices (`endpoint` list) in the template, so we could create one service instance that iterates over the `endpoint` list. 

```xml
<config-template xmlns="http://tail-f.com/ns/config/1.0"
                 servicepoint="ntp_server_example">
  <devices xmlns="http://tail-f.com/ns/ncs">
      <?foreach {/endpoint}?>
      <?set NTP_DEVICE={ntp-device}?>
      <?set NTP_REGION={region}?>
    <device>
      <name>{$NTP_DEVICE}</name>
      <config>
```

The `foreach` is referring to the `endpoint` list in the yang, and I also used the ability to set local variable names, just to illustrate the functionality `<?set NTP_DEVICE={ntp-device}?>`. So when the template runs, it will loop over each device and take the device name and set it to a new variable and set the associated region to a variable. 

### Conditionals

The template looks like as follows:

```xml
<config-template xmlns="http://tail-f.com/ns/config/1.0"
                 servicepoint="ntp_server_example">
  <devices xmlns="http://tail-f.com/ns/ncs">
      <?foreach {/endpoint}?>
      <?set NTP_DEVICE={ntp-device}?>
      <?set NTP_REGION={region}?>
    <device>
      <name>{$NTP_DEVICE}</name>
      <config>
        <!--
        AMER NTP Servers
        -->
        <?if {contains($NTP_REGION, 'AMER')}?>
          <ntp xmlns="urn:ios">
            <server>
              <peer-list>
                <name>10.20.30.40</name>
              </peer-list>
            </server>
          </ntp>

          <ntp xmlns="http://tail-f.com/ned/cisco-ios-xr">
            <server>
              <server-list>
                <name>10.20.30.40</name>
              </server-list>
            </server>
          </ntp>
          
          <ntp xmlns="http://tail-f.com/ned/cisco-nx">
            <server>
              <id>10.20.30.40</id>
            </server>
          </ntp>
         <?end?> <!-- end if -->
        <!--
        APJC NTP Servers
        -->
        <?if {contains($NTP_REGION, 'APJC')}?>
          <ntp xmlns="urn:ios">
            <server>
              <peer-list>
                <name>172.20.30.50</name>
              </peer-list>
            </server>
          </ntp>

          <ntp xmlns="http://tail-f.com/ned/cisco-ios-xr">
            <server>
              <server-list>
                <name>172.20.30.50</name>
              </server-list>
            </server>
          </ntp>
          
          <ntp xmlns="http://tail-f.com/ned/cisco-nx">
            <server>
              <id>172.20.30.50</id>
            </server>
          </ntp>
         <?end?> <!-- end if -->
        <!--
        EMEA NTP Servers
        -->
        <?if {contains($NTP_REGION, 'EMEA')}?>
          <ntp xmlns="urn:ios">
            <server>
              <peer-list>
                <name>192.20.30.60</name>
              </peer-list>
            </server>
          </ntp>
          
          <ntp xmlns="http://tail-f.com/ned/cisco-ios-xr">
            <server>
              <server-list>
                <name>192.20.30.60</name>
              </server-list>
            </server>
          </ntp>

          <ntp xmlns="http://tail-f.com/ned/cisco-nx">
            <server>
              <id>192.20.30.60</id>
            </server>
          </ntp>
         <?end?> <!-- end if -->
      </config>
    </device>
    <?end?> <!-- end foreach {/endpoint} -->
  </devices>
</config-template>
```

I used the XPATH function `contains()` to check to see if the region was one of our three options. You can see common XPATH functions [here](https://developer.mozilla.org/en-US/docs/Web/XPath/Functions). If you look at the sample output in this README above, you will notice that the first commit dry run does not include a region on purpose, to show you that the check is working and no config is created. 

Related to this discussion be careful of using a list key as a conditional check, [see this discussion](https://community.cisco.com/t5/nso-developer-hub-discussions/conditional-quot-if-quot-statement-is-not-working-as-expected/td-p/4019274)

