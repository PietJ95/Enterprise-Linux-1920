# Enterprise Linux Lab Report

- Student name: Piet Jacobs
- Github repo: <https://github.com/HoGentTIN/elnx-1920-sme-PietJ95.git>

Setting up basic automated installation for all upcoming servers

## Test plan

---
## Procedure/Documentation
## Router
Installing vagrant router plugin:

    vagrant plugin install vagrant-vyos

### Configuring router
This is straight forward thanks to the cheat sheet made by Bert Van Vreckem.  

#### Interfaces
Getting error on eth0:

    router: Can't configure static IPv4 address and DHCP on the same interface.
    router: [[interfaces ethernet eth0]] failed
    router: Commit failed
Fix was to not set up the NAT interface as static, this should be DHCP; sortaf was in the assignment description just looked over it.


#### DNS
Assignment: 
"All other DNS requests are forwarded to the appropriate IP address (the DNS server assigned by the "ISP"). Do not use Google's DNS servers or other public DNS resolvers!"  
-> Which ISP DNS server is this supposed to be?

#### Time zone & NTP
Ssh'd to the router to check the options in the terminal, found it to to be

    set system time-zone Europe/Brussels

NTP:
Thought I just needed to set the NTP server with this command:

    set system ntp server be.pool.ntp.org

But this is just the ip of the pool, every server in this pool needs to be added individually like this:

    set system ntp server 0.be.pool.ntp.org
    set system ntp server 1.be.pool.ntp.org
    set system ntp server 2.be.pool.ntp.org
    set system ntp server 3.be.pool.ntp.org
---
## DHCP
- Assign custom mac addresses to known hosts in the vagrant-hosts.yml file
- Make DHCP assign these mac addresses their specific IP

Subnet declaration:  
Having some issues with the subnet masks, giving me overlapping errors. Need to look further into this.

**Follow up**: I was actually trying to further subnet the internal network, but everything is supposed to be /24 and just further divided into pools for the DHCP.

This should be the working config:

    dhcp_subnets:
    #Guests
    - ip: 172.16.0.0
        netmask: 255.255.0.0
        range_begin: 172.16.0.2
        range_end: 172.16.127.254
        routers: 172.16.255.254
        default_lease_time: 14400 #4u

    #Workstations
    - ip: 172.16.0.0
        netmask: 255.255.0.0
        range_begin: 172.16.128.1
        range_end: 172.16.191.254
        routers: 172.16.255.254
        default_lease_time: 14400 #4u

    #Servers, gateway
    - ip: 172.16.0.0
        netmask: 255.255.0.0
        range_begin: 172.16.192.1
        range_end: 172.16.255.254
        routers: 172.16.255.254
        default_lease_time: 14400 #4u

## VM integration
All VM's need to have the correct default gateway set and the DNS servers in their network configuration  
Gateways:
- DMZ -> 192.0.2.254
- Internal -> 172.16.255.254

**DNS**: 172.16.192.1 & 172.16.192.2

Can't reach the HoGent DNS servers...
After troubleshooting with Van Maele A. I received feedback that my client DNS config should not be the pr001 DNS server but the router, it acts as a forwarder to all requests.

**So fix:** (Previous DNS config was wrong!)  
in /etc/resolv.conf
- DMZ machines: 192.0.2.254
- Internal machines: 172.16.255.254


## Make the traffic go through the network and not via NAT
What I wanted to do was just delete the NAT interface and add network routes but this would break vagrant connectivity in case I ever needed to provision again.

**Adding ansible groups for each subnet and setting their default gateway appropriately:**

    - hosts: internal
      become: true
      remote_user: root
      tasks:
        - name: Set default gateway with lowest metric
          shell: route add default gw 172.16.255.254 metric 99
        - name: Set dns servers
          shell: printf "nameserver 172.16.255.254" > /etc/resolv.conf

    - hosts: dmz
      become: true
      remote_user: root
      tasks:
        - name: Set default gateway
          shell: route add default gw 192.0.2.254 metric 99
        - name: Set dns servers
          shell: printf "nameserver 192.0.2.254" > /etc/resolv.conf

## Test report

Problems I encountered and their solution:  
- ...
    - Solution:  
    

- ...
    - Solution:   
    
## Resources
- BertVV VyOS cheat sheet
    - https://github.com/bertvv/cheat-sheets/blob/master/docs/VyOS.md
- https://support.vyos.io/en
- https://wiki.vyos.net/wiki/System_management#Configuring_date.2C_time_.26_NTP