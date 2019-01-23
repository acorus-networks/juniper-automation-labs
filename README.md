### This lab is based on Juniper Networks VQFX repo :
https://github.com/Juniper/vqfx10k-vagrant



This Vagrantfile will spawn 3 instances of VQFX (Full) and 1 ubuntu server

# Requirement

### Resources
 - RAM : 8G
 - CPU : 4 Cores

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

# Provisioning / Configuration

Both vqfx and srv will be preconfigured with Ip addresses


- vqfx: Ansible is used to configure interfaces
- srv: shell is used to configure interface eth1 with IP 192.168.100.10 
