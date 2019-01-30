# Juniper Workshop 2019 - By Acorus Networks

### This lab is based on Juniper Networks VQFX repo :
https://github.com/Juniper/vqfx10k-vagrant



# Requirement

This Vagrantfile will spawn 3 instances of VQFX (Full) and 1 ubuntu server

### Resources
 - RAM : 10G
 - CPU : 6 Cores

# Topology

                                                           =============
                                                           |   demo0X  |
                                                           =============   
                                                                 |
                                                                 |
                                                                 |       
                                                                 |  
                                         ------------------------------------------------                      
                                            |                                         |
                                         em0|                                      em0|
                                     =============  xe-0/0/4               eth1 =============
                    |----------------|           | --------------|------------- |   srv     |
                    |                |   vqfx1   |               |              =============
                    |       |------- |           |               |
                    |       |        =============               |
                    |       |            em1|                    |
                    |       |        =============               |
                    |       |        | vqfx-pfe1 |               |  
                    |       |        =============               |
                    |     xe-0/0/0                               |
                    |       |                                    |       
                    |       |                                    |
                    |       |                                    |   
                    |       |                                    |  
                    |       |                                    |            
                    |       |        =============  xe-0/0/4     |      
                    |       |------- |           | --------------|
                xe-0/0/1             |   vqfx2   |               |                     
                    |       |------- |           |               |  
                    |       |        =============               |   
                    |       |            em1|                    |   
                    |       |        =============               |   
                    |       |        | vqfx-pfe2 |               |   
                    |       |        =============               |   
                    |     xe-0/0/2                               |   
                    |       |                                    |   
                    |       |                                    |
                    |       |                                    |             
                    |       |                                    |
                    |       |                                    |          
                    |       |        =============  xe-0/0/4     |     
                    |       |------- |           | --------------|
                    |                |   vqfx3   |              
                    |----------------|           |
                                     =============
                                         em1|
                                     =============
                                     | vqfx-pfe3 |
                                     =============

# Provisioning

All VQFX and server will be preconfigured.


- vqfx: Ansible is used to configure interfaces
- srv: Ansible is used to prepare the machine (packages, ansible and demo files)

## Tools

- Ansible 
- Vagrant / Virtualbox
- Bird (installed on the server)
- Ansible Vault

## Installation

#### SSH on the demo machine :

```
$ ssh acorus@your_pod_ip
```

#### Check vagrant env (shoud be ready to use) :

```
acorus@demo01:~$ cd juniper-automation-labs
acorus@demo01:~/juniper-automation-labs$ vagrant status
```

#### SSH to devices on lab :

Open 3 more terminal, connnect to your POD demo server in each. You should have 4 terminal opened and connected to your demo server now.
Then SSH to each device in the lab, repeat for vqfx1, vqfx2, vqfx3, srv.

```
acorus@demo01:~$ cd juniper-automation-labs
acorus@demo01:~/juniper-automation-labs$ vagrant ssh vqfx1
```

#### Test on VQFX1 :

Run below command on vqfx1 :

```
vagrant@vqfx1> show configuration
```

OUTPUT

```
vagrant@vqfx1> show configuration
## Last commit: 2019-01-24 12:34:12 UTC by root
version 17.4R1.16;
system {
    host-name vqfx1;
    root-authentication {
        encrypted-password
...
```

#### Test on SRV :

Run below command on srv :

```
vagrant@server:~$ ifconfig
vagrant@server:~$ ping 192.168.100.20
vagrant@server:~$ ssh vqfx1
```
You should know be able to SSH lab VQFX from srv (192.168.100.10) under noc user :

OUTPUT
 
```
vagrant@server:~$ ssh vqfx1
--- JUNOS 17.4R1.16 built 2017-12-19 20:03:37 UTC
{master:0}
noc@vqfx1>
```


Let's try with Ansible now :

```
vagrant@server$ ansible-playbook -i inventories/hosts pb.test.netconf.yaml --vault-id ~/.vault_pass.txt
```

OUTPUT

```
vagrant@server:~$ ansible-playbook -i inventories/hosts pb.test.netconf.yaml --vault-id ~/.vault_pass.txt

PLAY [Compare  Junos OS] *****************************************************************************************************************************

TASK [Checking NETCONF connectivity] *****************************************************************************************************************
ok: [vqfx2]
ok: [vqfx3]
ok: [vqfx1]

TASK [Print IP of remote device] *********************************************************************************************************************
ok: [vqfx1] => {
    "msg": "192.168.100.20"
}
ok: [vqfx2] => {
    "msg": "192.168.100.21"
}
ok: [vqfx3] => {
    "msg": "192.168.100.22"
}

TASK [Retrieving information from devices running Junos OS] ******************************************************************************************
ok: [vqfx2]
ok: [vqfx3]
ok: [vqfx1]

TASK [Print version] *********************************************************************************************************************************
ok: [vqfx1] => {
    "junos.version": "17.4R1.16"
}
ok: [vqfx2] => {
    "junos.version": "17.4R1.16"
}
ok: [vqfx3] => {
    "junos.version": "17.4R1.16"
}

PLAY RECAP *******************************************************************************************************************************************
vqfx1                      : ok=4    changed=0    unreachable=0    failed=0
vqfx2                      : ok=4    changed=0    unreachable=0    failed=0
vqfx3                      : ok=4    changed=0    unreachable=0    failed=0
```
Everything is tested and ready, let's start the lab now !



# Labs

### Step 1 : Add a new user on all network devices

In this lab we'll show how to create and deploy new users on a group of network devices.

It a good practice to update allowed users/password on a regular basis.

Automation comes really handy when the number of managed devices increase.

For this lab we'll also use Ansible vault feature to store these confidential informations.


#### Create a new user locally on VQFX1 to generate user HASH

We'll use JunoOS for that, connect to VQFX1 :

```
vagrant@vqfx1> edit
vagrant@vqfx1# set system login user {YOUR_USER} class read-only authentication plain-text-password
New password:
Retype new password:
```

We'll use the generated hash (starts with $6$) :

```
vagrant@vqfx1# show | compare
```

OUTPUT

```
[edit system login]
+    user acorso {
+        class read-only;
+        authentication {
+            encrypted-password "$6$cswP1RmP$HuNkD7BrUTZqNTqs1oPTc4vmpJFHUbydfNymjg86OWtlr8gnjn9Y0hW3HBf6VzSuDbpE7yZTD.tc1b49LXb8G0"; ## SECRET-DATA
+        }
+    }

```

Copy the hash for the next step, now discard the pending changes :

```
{master:0}[edit]
vagrant@vqfx1# rollback
load complete

{master:0}[edit]
vagrant@vqfx1# exit
Exiting configuration mode
```


#### Edit inventory to create the new user

Add your user in users file :

```
vagrant@server$ ansible-vault edit inventories/group_vars/all/users.yaml
Vault password:
```

Password is  : ```gN4PFTz5nmWWqpiYzmrcdSwHif6ePkYy7zwyhfkeFAQW9HwAVM3edKjDAmM5nKa```

Append to the file :

```
  - name: YOUR_USER
    fullname: "Insert description here"
    class: "acorusread"
    password: "HASH_TO_COPY"
```


#### Push the configuration to all devices

```
vagrant@server$ ansible-playbook -i inventories/hosts pb.juniper.users.yaml --vault-id ~/.vault_pass.txt
```

OUTPUT

```
PLAY [Config Users of Juniper devices] ***********************************************************************************************************************************

TASK [base : Create host specific configuration directory] ***************************************************************************************************************
changed: [vqfx2]
changed: [vqfx1]
changed: [vqfx3]

TASK [base : file] *******************************************************************************************************************************************************
ok: [vqfx1]
ok: [vqfx2]
ok: [vqfx3]

TASK [base : file] *******************************************************************************************************************************************************
ok: [vqfx1]
ok: [vqfx2]
ok: [vqfx3]

TASK [base : Base Configuration] *****************************************************************************************************************************************
changed: [vqfx2] => (item=users.conf)
changed: [vqfx1] => (item=users.conf)
changed: [vqfx3] => (item=users.conf)

TASK [Assembling configurations and copying to conf] *********************************************************************************************************************
changed: [vqfx2]
changed: [vqfx1]
changed: [vqfx3]

TASK [Pushing config ... please wait ...] ********************************************************************************************************************************
changed: [vqfx2]
changed: [vqfx1]
changed: [vqfx3]

TASK [Print the difference if exists] ************************************************************************************************************************************
ok: [vqfx1] => {
    "response.diff_lines": [
        "",
        "[edit system login]",
        "+    user ansible-bot {",
        "+        full-name \"Ansible bot for automation\";",
        "+        uid 2002;",
        "+        class super-user;",                  : ok=7    changed=3    unreachable=0    failed=0

```

#### Verification 

On VQFX1 :

```
vagrant@vqfx1> show configuration system login user {YOUR_USER}
```

OUTPUT

```
full-name "xxxx";
uid 2006;
class acorusread;
authentication {
    encrypted-password "$6$GAMRvF.5$.yhfl02NtcTn03p.9Z/Mts0Rhhif0qibapCXdlMa8DvzfGidRknsFQRVH6VTMUyc4DST9.hWZoB5EjE0D3cdC/"; ## SECRET-DATA
}
```

Try to ssh from server with the new user (use IP address of one of the VQFX as we already have entries in .ssh/config file) :

```
vagrant@server:~$ ssh {YOUR_USER}@192.168.100.20
Password:
```

Login should succeed and your new user now have a limited access on the device, show & ping. As you can see edit mode isn't available.


```
{YOUR_USER}@vqfx1> edit
```

OUTPUT

```
            ^
unknown command.

{master:0}
```

Now try some show commands :

```
{YOUR_USER}@vqfx1> show configuration system
```

OUTPUT


```
host-name vqfx1;
root-authentication {
    encrypted-password /* SECRET-DATA */; ## SECRET-DATA
    ssh-rsa /* SECRET-DATA */;
}
login {
    class acorusread {
        idle-timeout 15;
        permissions [ view view-configuration ];
        allow-commands "(ping.*)|(show.*)|(exit)";
    }
    user ansible-bot {
        full-name "Ansible bot for automation";
```


### Step 2: IGP configuration

In this lab we'll configure OSPF to perform loopbacks reachability.

It is a requirement for the next lab as we'll setup IBGP sessions.

We'll also run some tests to ensure we have a working topology.

#### Configure loopbacks on devices :

Current setup :

```
vagrant@vqfx1> show ospf overview
OSPF instance is not running
```


### Step 3: Configure BGP sessions

Explain lab ...


#### Establish IBGP sessions

Current setup :

```
vagrant@vqfx1> show bgp summary
BGP is not running
```

Template is already created, check it then apply it.

```
vagrant@server$ ansible-playbook -i inventories/hosts pb.juniper.bgp.yaml --vault-id ~/.vault_pass.txt
```

OUTPUT

```
TASK [Print the difference if exists] ***********************************************************************************************
ok: [vqfx1] => {
    "response.diff_lines": [
        "",
        "[edit protocols]",
        "+   bgp {",
        "+       group ipv4-transit-123 {",
        "+           type external;",
        "+           description \"IPV4 AS123\";",
        "+           local-address 10.1.2.1;",
        "+           import ipv4-as123;",
        "+           family inet {",
        "+               unicast;",
        "+           }",
        "+           peer-as 123;",
        "+           local-as 65500;",
        "+           neighbor 10.1.2.2;",
        "+       }",
        "+   }",
        "[edit]",
        "+  policy-options {",
        "+      policy-statement ipv4-as123 {",
        "+          then accept;"
        "+      }",
        "+  }"
    ]
}

PLAY RECAP **************************************************************************************************************************
vqfx1                       : ok=7    changed=4    unreachable=0    failed=0
```

#### Verification

Session BGP UP

```
vagrant@vqfx1> show bgp summary
```

Received routes

```
vagrant@vqfx1> show route protocol bgp
```

OUTPUT

```
inet.0: 9 destinations, 9 routes (9 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

1.0.0.0/24         *[BGP/170] 00:02:26, localpref 100
                      AS path: 123 I, validation-state: unverified
                    > to 10.1.2.2 via xe-0/0/0.0
2.0.0.0/24         *[BGP/170] 00:02:48, localpref 100
                      AS path: 123 I, validation-state: unverified
                    > to 10.1.2.2 via xe-0/0/0.0
```

### Step 4: Edit imports on BGP sessions

Ajouter des variables dans l'iventaire pour autoriser seulement 1 prefixe

```
vagrant@server:~/ansible$ nano inventories/group_vars/all/bgp_transit_filter.yaml
```

Le format est le suivant, on autorise seulement le prefix 2.0.0.0/24 a etre annonce par l'AS 123

```
---
transit_filtered_prefixes:
  123:
    ipv4:
    - 2.0.0.0/24
```

On pousse la modification via ansible

```
vagrant@server:~/ansible$ ansible-playbook -i inventories/hosts pb.juniper.bgp.yaml --vault-id ~/.vault_pass.txt
```

OUTPUT

```
TASK [Print the difference if exists] ***********************************************************************************************
ok: [vqfx] => {
    "response.diff_lines": [
        "",
        "[edit policy-options policy-statement ipv4-as123]",
        "+    term accepted {",
        "+        from {",
        "+            route-filter 2.0.0.0/24 exact;",
        "+        }",
        "+        then accept;",
        "+    }",
        "+    term other {",
        "+        then reject;",
        "+    }",
        "[edit policy-options policy-statement ipv4-as123]",
        "-    then accept;"
    ]
}
```

#### Verification

On peut verifier qu'il n'y a plus qu'une route acceptée

```
vagrant@vqfx-re> show route receive-protocol bgp 10.1.2.2
```

OUTPUT

```
inet.0: 9 destinations, 9 routes (8 active, 0 holddown, 1 hidden)
  Prefix      Nexthop        MED     Lclpref    AS path
* 2.0.0.0/24              10.1.2.2                                123 I
```

On peut aussi voir la route refusée

```
vagrant@vqfx-re> show route receive-protocol bgp 10.1.2.2 hidden
```

OUTPUT

```
inet.0: 9 destinations, 9 routes (8 active, 0 holddown, 1 hidden)
  Prefix      Nexthop        MED     Lclpref    AS path
  1.0.0.0/24              10.1.2.2                                123 I
```

### Etape 4: Supprimer un filtre

Find a solution :)
