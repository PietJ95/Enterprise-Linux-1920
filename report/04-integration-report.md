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

## Test report

Problems I encountered and their solution:  
- ...
    - Solution:  
    

- ...
    - Solution:   
    
## Resources
BertVV VyOS cheat sheet
- https://github.com/bertvv/cheat-sheets/blob/master/docs/VyOS.md
- https://support.vyos.io/en
- https://wiki.vyos.net/wiki/System_management#Configuring_date.2C_time_.26_NTP