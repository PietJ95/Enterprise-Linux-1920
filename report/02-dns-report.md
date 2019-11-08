# Enterprise Linux Lab Report

- Student name: Piet Jacobs
- Github repo: <https://github.com/HoGentTIN/elnx-1920-sme-PietJ95.git>

Setting up basic automated installation for all upcoming servers

## Test plan

Issues getting the slave to load in the remote zone file...



## Procedure/Documentation
### Master
Basic DNS config is pretty straight forward.

### Slave
Slave was failing to get the zone file from the master:
- Did some double checking in /etc/named.conf to see if everything was correct. Found no mistakes.
- Tried using additional role vars such as "also-notify", "allow-update". Did not help
- Asked mr Van Maele for help, pointed me to the firewall
    - Did not think of this myself, since I must have misread the role and thought that firewall wouldn't be an issue.

**The fix was:**

    sudo firewall-cmd --zone=public --add-service=dns --permanent

More tests are passing now so the zone file is transferring

    ✗ Reverse lookups public servers (from function `assert_reverse_lookup' in file /vagrant/test/pr002/slavedns.bats, line 28, in test file /vagrant/test/pr002/slavedns.bats, line 122)
    `assert_reverse_lookup pu001      192.0.2.10' failed
    ✗ Reverse lookups private servers
    (from function `assert_reverse_lookup' in file /vagrant/test/pr002/slavedns.bats, line 28, in test file /vagrant/test/pr002/slavedns.bats, line 128)
    `assert_reverse_lookup pr001      172.16.192.1' failed
    ✓ Alias lookups public servers
    ✓ Alias lookups private servers
    ✓ NS record lookup
    ✓ Mail server lookup


The reverse lookups are failing...
## Test report


Problems I encountered and their solution:  
- ...
    - Solution:  
    

- ...
    - Solution:   
    
## Resources

