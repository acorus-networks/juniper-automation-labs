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
- Ansible Vault

## Lab setup

#### SSH on the demo machine :

```
$ ssh acorus@{{your_pod_ip}}
```

#### Check vagrant env (shoud be ready to use) :

```
acorus@demo01:~$ cd juniper-automation-labs
acorus@demo01:~/juniper-automation-labs$ vagrant status
```

#### SSH to devices on lab :

Open 3 more terminals, connnect to your POD demo server in each. You should have 4 terminal opened and connected to your demo server now.
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

Run below command on the srv :

```
vagrant@server:~$ ifconfig
vagrant@server:~$ ping 192.168.100.20
vagrant@server:~$ ssh vqfx1
```
You should be able to SSH lab VQFX from srv (192.168.100.10) under noc user :

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

## Step 1 : Add a new user on all network devices

In this lab we'll show how to create and deploy new users on a group of network devices.

It a good practice to update allowed users/password on a regular basis.

Automation comes really handy when the number of managed devices increase.

For this lab we'll also use Ansible vault feature to store these confidential informations.


#### Create a new user locally on VQFX1 to generate user HASH

We'll use JunoOS for that, connect to VQFX1 :

```
vagrant@vqfx1> edit
vagrant@vqfx1# set system login user {{YOUR_USER}} class read-only authentication plain-text-password
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

The VAULT password is  : ```gN4PFTz5nmWWqpiYzmrcdSwHif6ePkYy7zwyhfkeFAQW9HwAVM3edKjDAmM5nKa```

Append to the file :

```
  - name: {{YOUR_USER}}
    fullname: "{{Insert description here}}"
    class: "acorusread"
    password: "{{HASH_TO_COPY}}"
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
        "+        class super-user;", 
```

#### Verification 

On VQFX1 :

```
vagrant@vqfx1> show configuration system login user {{YOUR_USER}}
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
vagrant@server:~$ ssh {{YOUR_USER}}@192.168.100.20
Password:
```

Login should succeed and your new user now have a limited access on the device, show & ping. As you can see edit mode isn't available.


```
{YOUR_USER}@vqfx1> edit
```

OUTPUT

```
{YOUR_USER}@vqfx1> edit
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


## Step 2: IGP configuration

In this lab we'll configure interco's and OSPF to perform loopbacks reachability.

It is a requirement for the next lab as we'll setup IBGP sessions.

We'll also run some tests to ensure we have a working topology.

#### Templates review & configuration :

Current setup :

```
vagrant@vqfx1> show ospf overview
OSPF instance is not running
```

Check content of ```~/inventories/host_vars/vqfx1/igp.yaml```
Check content of ```~/roles/igp/templates/*.yaml```

The playbook below will configure interfaces, loopbacks & OSPF.

```
ansible-playbook -i inventories/hosts pb.juniper.igp.yaml --vault-id ~/.vault_pass.txt
```

New setup :

```
vagrant@vqfx1> show ospf overview
Instance: master
  Router ID: 10.200.0.1
  Route table index: 0
  LSA refresh time: 50 minutes
  Area: 0.0.0.0
    Stub type: Not Stub
    Authentication Type: None
    Area border routers: 0, AS boundary routers: 0
    Neighbors
      Up (in full state): 2
  Topology: default (ID 0)
    Prefix export count: 0
    Full SPF runs: 7
    SPF delay: 0.200000 sec, SPF holddown: 5 sec, SPF rapid runs: 3
    Backup SPF: Not Needed

{master:0}
```

#### Let's run some tests :

Now we'll run a playbook to ping all our directly connected neighbors and their loopbacks, igp.yaml file will used to get ips of peers.

```
ansible-playbook -i inventories/hosts pb.juniper.ping.yaml --vault-id ~/.vault_pass.txt

```
OUTPUT

```
vagrant@server:~$ ansible-playbook -i inventories/hosts pb.juniper.ping.yaml --vault-id ~/.vault_pass.txt

PLAY [Ping devices] *********************************************************************************************************************

TASK [check if neighbors are reacheable on intercos] ************************************************************************************
ok: [vqfx1] => (item={u'interface': u'xe-0/0/0', u'peer': u'vqfx2', u'vlans': [{u'id': 0, u'ipv4': {u'netmask': 24, u'local_ip': u'192.168.2.1', u'peer_ip': u'192.168.2.2'}}], u'speed': u'10G', u'type': u'local-core'})
ok: [vqfx2] => (item={u'interface': u'xe-0/0/0', u'peer': u'vqfx1', u'vlans': [{u'id': 0, u'ipv4': {u'netmask': 24, u'local_ip': u'192.168.2.2', u'peer_ip': u'192.168.2.1'}}], u'speed': u'10G', u'type': u'local-core'})
ok: [vqfx3] => (item={u'interface': u'xe-0/0/1', u'peer': u'vqfx1', u'vlans': [{u'id': 0, u'ipv4': {u'netmask': 24, u'local_ip': u'192.168.3.2', u'peer_ip': u'192.168.3.1'}}], u'speed': u'10G', u'type': u'local-core'})
ok: [vqfx1] => (item={u'interface': u'xe-0/0/1', u'peer': u'vqfx3', u'vlans': [{u'id': 0, u'ipv4': {u'netmask': 24, u'local_ip': u'192.168.3.1', u'peer_ip': u'192.168.3.2'}}], u'speed': u'10G', u'type': u'local-core'})
ok: [vqfx2] => (item={u'interface': u'xe-0/0/2', u'peer': u'vqfx3', u'vlans': [{u'id': 0, u'ipv4': {u'netmask': 24, u'local_ip': u'192.168.4.1', u'peer_ip': u'192.168.4.2'}}], u'speed': u'10G', u'type': u'local-core'})
ok: [vqfx3] => (item={u'interface': u'xe-0/0/2', u'peer': u'vqfx2', u'vlans': [{u'id': 0, u'ipv4': {u'netmask': 24, u'local_ip': u'192.168.4.2', u'peer_ip': u'192.168.4.1'}}], u'speed': u'10G', u'type': u'local-core'})

TASK [check if neighbors loopbacks are reacheable] **************************************************************************************
ok: [vqfx1] => (item=10.200.0.1)
ok: [vqfx2] => (item=10.200.0.1)
ok: [vqfx3] => (item=10.200.0.1)
ok: [vqfx1] => (item=10.200.0.2)
ok: [vqfx2] => (item=10.200.0.2)
ok: [vqfx3] => (item=10.200.0.2)
ok: [vqfx1] => (item=10.200.0.3)
ok: [vqfx2] => (item=10.200.0.3)
ok: [vqfx3] => (item=10.200.0.3)

PLAY RECAP ******************************************************************************************************************************
vqfx1                      : ok=2    changed=0    unreachable=0    failed=0
vqfx2                      : ok=2    changed=0    unreachable=0    failed=0
vqfx3                      : ok=2    changed=0    unreachable=0    failed=0
```


You should have loopbacks of all VQFX in the routing table now (run the command on vqfx1):

```
vagrant@vqfx1> show route 10.200.0.0/24
```

OUTPUT

```
inet.0: 16 destinations, 17 routes (16 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.200.0.1/32      *[Direct/0] 01:09:39
                    > via lo0.0
10.200.0.2/32      *[OSPF/10] 00:13:52, metric 2500
                    > to 192.168.2.2 via xe-0/0/0.0
10.200.0.3/32      *[OSPF/10] 00:13:44, metric 2500
                    > to 192.168.3.2 via xe-0/0/1.0
```

Let's try to shut the link between VQFX1 and VQFX3 :

```
vagrant@vqfx1> edit
vagrant@vqfx1# set interfaces xe-0/0/1 disable
vagrant@vqfx1# commit
```

Re-run the test playbook and check that all loopbacks are still reacheable from all devices :

```
ansible-playbook -i inventories/hosts pb.juniper.ping.yaml --vault-id ~/.vault_pass.txt

```
OUTPUT

```
vagrant@server:~$ ansible-playbook -i inventories/hosts pb.juniper.ping.yaml --vault-id ~/.vault_pass.txt

PLAY [Ping devices] *********************************************************************************************************************

TASK [check if neighbors are reacheable on intercos] ************************************************************************************
ok: [vqfx2] => (item={u'interface': u'xe-0/0/0', u'peer': u'vqfx1', u'vlans': [{u'id': 0, u'ipv4': {u'netmask': 24, u'local_ip': u'192.168.2.2', u'peer_ip': u'192.168.2.1'}}], u'speed': u'10G', u'type': u'local-core'})
ok: [vqfx1] => (item={u'interface': u'xe-0/0/0', u'peer': u'vqfx2', u'vlans': [{u'id': 0, u'ipv4': {u'netmask': 24, u'local_ip': u'192.168.2.1', u'peer_ip': u'192.168.2.2'}}], u'speed': u'10G', u'type': u'local-core'})
failed: [vqfx3] (item={u'interface': u'xe-0/0/1', u'peer': u'vqfx1', u'vlans': [{u'id': 0, u'ipv4': {u'netmask': 24, u'local_ip': u'192.168.3.2', u'peer_ip': u'192.168.3.1'}}], u'speed': u'10G', u'type': u'local-core'}) => {"acceptable_percent_loss": "0", "changed": false, "count": "5", "dest": "192.168.3.1", "dest_ip": "192.168.3.1", "do_not_fragment": false, "host": "192.168.3.1", "interface": null, "item": {"interface": "xe-0/0/1", "peer": "vqfx1", "speed": "10G", "type": "local-core", "vlans": [{"id": 0, "ipv4": {"local_ip": "192.168.3.2", "netmask": 24, "peer_ip": "192.168.3.1"}}]}, "msg": "Loss 100%, (Sent 5 | Received 0)", "packet_loss": "100", "packets_received": "0", "packets_sent": "5", "rapid": true, "routing_instance": null, "rtt_average": null, "rtt_maximum": null, "rtt_minimum": null, "rtt_stddev": null, "size": null, "source": "192.168.3.2", "source_ip": "192.168.3.2", "state": "absent", "timeout": "30", "ttl": "1"}
ok: [vqfx2] => (item={u'interface': u'xe-0/0/2', u'peer': u'vqfx3', u'vlans': [{u'id': 0, u'ipv4': {u'netmask': 24, u'local_ip': u'192.168.4.1', u'peer_ip': u'192.168.4.2'}}], u'speed': u'10G', u'type': u'local-core'})
ok: [vqfx3] => (item={u'interface': u'xe-0/0/2', u'peer': u'vqfx2', u'vlans': [{u'id': 0, u'ipv4': {u'netmask': 24, u'local_ip': u'192.168.4.2', u'peer_ip': u'192.168.4.1'}}], u'speed': u'10G', u'type': u'local-core'})
...ignoring
 [WARNING]: sendto: Can't assign requested address

failed: [vqfx1] (item={u'interface': u'xe-0/0/1', u'peer': u'vqfx3', u'vlans': [{u'id': 0, u'ipv4': {u'netmask': 24, u'local_ip': u'192.168.3.1', u'peer_ip': u'192.168.3.2'}}], u'speed': u'10G', u'type': u'local-core'}) => {"acceptable_percent_loss": "0", "changed": false, "count": "5", "dest": "192.168.3.2", "dest_ip": "192.168.3.2", "do_not_fragment": false, "host": "192.168.3.2", "interface": null, "item": {"interface": "xe-0/0/1", "peer": "vqfx3", "speed": "10G", "type": "local-core", "vlans": [{"id": 0, "ipv4": {"local_ip": "192.168.3.1", "netmask": 24, "peer_ip": "192.168.3.2"}}]}, "msg": "Loss 100%, (Sent 5 | Received 0)", "packet_loss": "100", "packets_received": "0", "packets_sent": "5", "rapid": true, "routing_instance": null, "rtt_average": null, "rtt_maximum": null, "rtt_minimum": null, "rtt_stddev": null, "size": null, "source": "192.168.3.1", "source_ip": "192.168.3.1", "state": "absent", "timeout": "30", "ttl": "1"}
...ignoring

TASK [check if neighbors loopbacks are reacheable] **************************************************************************************
ok: [vqfx1] => (item=10.200.0.1)
ok: [vqfx2] => (item=10.200.0.1)
ok: [vqfx3] => (item=10.200.0.1)
ok: [vqfx1] => (item=10.200.0.2)
ok: [vqfx2] => (item=10.200.0.2)
ok: [vqfx3] => (item=10.200.0.2)
ok: [vqfx3] => (item=10.200.0.3)
ok: [vqfx2] => (item=10.200.0.3)
ok: [vqfx1] => (item=10.200.0.3)

```

We can confirm that interco vqfx1 <--> vqfx3 is unreachable. We even have a warning on vqfx1 (sendto: Can't assign requested address), this is the interface we disabled.
On the other hand all our loopbacks are still online. 

```
vagrant@vqfx1> show route 10.200.0.0/24
```

OUTPUT

```
vagrant@vqfx1> show route 10.200.0.0/24

inet.0: 16 destinations, 17 routes (16 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.200.0.1/32      *[Direct/0] 3d 20:52:54
                    > via lo0.0
10.200.0.2/32      *[OSPF/10] 3d 20:52:34, metric 2500
                    > to 192.168.2.2 via xe-0/0/0.0
10.200.0.3/32      *[OSPF/10] 00:12:14, metric 5000
                    > to 192.168.2.2 via xe-0/0/0.0
```

Note the increased metric on 10.200.0.3/32 route from vqfx routing table.

Last step re-activate the disabled interface on vqfx1 :

```
vagrant@vqfx1> edit
vagrant@vqfx1# delete interfaces xe-0/0/1 disable
vagrant@vqfx1# commit
```
 

## Step 3: Configure BGP sessions

In this lab we'll configure iBGP sessions, full mesh.

In this step bgp sessions, but also policies will be added to the config.

Our AS for this lab will be 65500.

We also generate a dummy prefix on each vqfx (for testing purpose) :

vqfx1 = 1.0.0.0/8

vqfx2 = 2.0.0.0/8

vqfx3 = 3.0.0.0/8

Generated prefixes will be advertised (via send-static policy).


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
vagrant@server:~$ ansible-playbook -i inventories/hosts pb.juniper.bgp.yaml --vault-id ~/.vault_pass.txt

PLAY [Config BGP of Juniper devices] ****************************************************************************************************

...

TASK [Pushing config ... please wait ...] ***********************************************************************************************
changed: [vqfx2]
changed: [vqfx3]
changed: [vqfx1]

TASK [Print the difference if exists] ***************************************************************************************************
ok: [vqfx1] => {
    "response.diff_lines": [
        "",
        "[edit routing-options]",
        "+   static {",
        "+       route 1.0.0.0/8 {",
        "+           discard;",
        "+           install;",
        "+           readvertise;",
        "+           active;",
        "+       }",
        "+   }",
        "[edit protocols]",
 
```

#### Checks

You should have two peers with established sessions :

```
vagrant@vqfx1> show bgp summary
```

OUTPUT

```
vagrant@vqfx1> show bgp summary
Groups: 2 Peers: 2 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0
                       2          2          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.200.0.2            65500         10          9       0       0        3:15 1/1/1/0              0/0/0/0
10.200.0.3            65500         10          9       0       0        3:17 1/1/1/0              0/0/0/0
```


Look at the received routes :

```
vagrant@vqfx1> show route protocol bgp
```

OUTPUT

```
vagrant@vqfx1> show route protocol bgp

inet.0: 19 destinations, 20 routes (19 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

2.0.0.0/8          *[BGP/170] 00:04:05, localpref 100, from 10.200.0.2
                      AS path: I, validation-state: unverified
                    > to 192.168.2.2 via xe-0/0/0.0
3.0.0.0/8          *[BGP/170] 00:04:07, localpref 100, from 10.200.0.3
                      AS path: I, validation-state: unverified
                    > to 192.168.3.2 via xe-0/0/1.0
```

#### Test BGP state with a playbook :

Create a playbook to check if sessions on all vqfx are established.

You can re-use the syntax of pb.juniper.ping.yaml.

Your playbook should give the below output :

OUTPUT

```
vagrant@server:~$ ansible-playbook -i inventories/hosts pb.juniper.check-bgp.yaml --vault-id ~/.vault_pass.txt

PLAY [Check BGP] ****************************************************************************************************************************************************

TASK [check bgp peers states] ***************************************************************************************************************************************
ok: [vqfx3] => (item={u'neighbor_ipv4': u'10.200.0.1', u'local_ipv4': u'10.200.0.3', u'asn': 65500, u'name': u'ibgp-vqfx1'})
ok: [vqfx1] => (item={u'neighbor_ipv4': u'10.200.0.2', u'local_ipv4': u'10.200.0.1', u'asn': 65500, u'name': u'ibgp-vqfx2'})
ok: [vqfx2] => (item={u'neighbor_ipv4': u'10.200.0.1', u'local_ipv4': u'10.200.0.2', u'asn': 65500, u'name': u'ibgp-vqfx1'})
ok: [vqfx2] => (item={u'neighbor_ipv4': u'10.200.0.3', u'local_ipv4': u'10.200.0.2', u'asn': 65500, u'name': u'ibgp-vqfx3'})
ok: [vqfx3] => (item={u'neighbor_ipv4': u'10.200.0.2', u'local_ipv4': u'10.200.0.3', u'asn': 65500, u'name': u'ibgp-vqfx2'})
ok: [vqfx1] => (item={u'neighbor_ipv4': u'10.200.0.3', u'local_ipv4': u'10.200.0.1', u'asn': 65500, u'name': u'ibgp-vqfx3'})
```


## Step 4: Filter imports on BGP sessions

In this step we'll automate the process of denying a specific prefix in import policies of peers.

For this lab you will have to find a solution by yourself. 

We want prefix 2.0.0.0/8 to be denied by all import policies of all peers.


#### Add vars with denied prefix

Have a look at the templates in the BGP role, it will give you clues on how to deny the prefix.

Create a new variables file in inventory to discard one prefix to be imported by policies.

#### Push the config

Re-run the bgp playbook to push your changes.

#### Checks

The denied prefix (2.0.0.0/8) is absent from vqfx1 routing table :

```
vagrant@vqfx1> show route protocol bgp
```

OUTPUT

```
vagrant@vqfx1> show route protocol bgp

inet.0: 19 destinations, 20 routes (18 active, 0 holddown, 1 hidden)
+ = Active Route, - = Last Active, * = Both

3.0.0.0/8          *[BGP/170] 00:33:22, localpref 100, from 10.200.0.3
                      AS path: I, validation-state: unverified
                    > to 192.168.3.2 via xe-0/0/1.0

```

We have one hidden route, let's check it :


```
vagrant@vqfx1> show route protocol bgp hidden
```

OUTPUT

```
inet.0: 19 destinations, 20 routes (18 active, 0 holddown, 1 hidden)
+ = Active Route, - = Last Active, * = Both

2.0.0.0/8           [BGP ] 00:12:37, localpref 100, from 10.200.0.2
                      AS path: I, validation-state: unverified
                    > to 192.168.2.2 via xe-0/0/0.0
```

Finally look at the bgp summary command :

```
vagrant@vqfx1> show bgp summary
```

OUTPUT

```
Groups: 2 Peers: 2 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0
                       2          1          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.200.0.2            65500         84         83       0       0       36:20 0/1/0/0              0/0/0/0
10.200.0.3            65500         75         82       0       0       36:22 1/1/1/0              0/0/0/0
```

We see that zero routes are imported on peer 10.200.0.2 (below state row).

#### Restore routing

Now find a way to remove the filtering on your peers.


## Step 5: Adapt local preference

For this lab you have to change the local preference for 3.0.0.0/8 towards 10.200.0.3 on import policy of vqfx1 to the value of 500.

Of course without breaking the previous filtering system you deployed in step 4.

Feel free to find any solution :)

Check the pro tips below in case you break something on the lab.

## Pro tips : 

If you want to reset the to the orginal state, run :

```
acorus@demo1:~/juniper-automation-labs$ vagrant provision
```

Rollback configuration to a previous state on vqfx, first select commit to rollback :

```
vagrant@vqfx1# show | compare rollback 1
[edit groups]
-  /* OSPF Core */
-  ospf-core {
-      protocols {
-          ospf {
```

Perform rollback to the desired commit number :

```
vagrant@vqfx1# rollback 1
load complete

{master:0}[edit]
vagrant@vqfx1# commit
```
