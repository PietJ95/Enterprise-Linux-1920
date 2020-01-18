# Enterprise Linux Lab Report

- Student name: Piet Jacobs
- Github repo: <https://github.com/HoGentTIN/elnx-1920-sme-PietJ95.git>

Setting up basic automated installation for all upcoming servers

## Test plan
### Verify tests
**`pr001 (Master)`**
- Open a terminal in the root directory of this project
- Bring up the machine with `vagrant up pr001`
- Connect to the machine with `vagrant ssh pr001`
- Run the following command and verify that all tests pass

        sudo /vagrant/test/runbats.sh

**`pr002 (Slave)`**
- Open a terminal in the root directory of this project
- Bring up the machine with `vagrant up pr002`
- Connect to the machine with `vagrant ssh pr002`
- Run the following command and verify that all tests pass
    
        sudo /vagrant/test/runbats.sh
        
### host connectivity
Use `nslookup` to verify the domain:

    #pr001
    nslookup avalon.lan 172.16.192.1   

    #pr002
    nslookup avalon.lan 172.16.192.2 

**This should return the server pr001/pr002.avalon.lan with the IP**



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

Checked the directory on pr002 to see which zones were being loaded. So the reverse lookup zones are defined by

    - networks:
      - '172.16'
      - '192.0.2'
These weren't defined in the slave configuration, so basically just needed to add the same thing under the 'avalon.lan' domain to make these reverse lookup zones replicate aswell.

## Test report

- pr001: verifying tests

        Running test /vagrant/test/common.bats
        ✓ SELinux should be set to 'Enforcing'
        ✓ Firewall should be enabled and running
        ✓ EPEL repository should be available
        ✓ Bash-completion should have been installed
        ✓ bind-utils should have been installed
        ✓ Git should have been installed
        ✓ Nano should have been installed
        ✓ Tree should have been installed
        ✓ Vim-enhanced should have been installed
        ✓ Wget should have been installed
        ✓ Admin user piet should exist
        ✓ An SSH key should have been installed for piet
        ✓ Custom /etc/motd should have been installed

        13 tests, 0 failures
        Running test /vagrant/test/pr001/masterdns.bats
        ✓ The dig command should be installed
        ✓ The main config file should be syntactically correct
        ✓ The forward zone file should be syntactically correct
        ✓ The reverse zone files should be syntactically correct
        ✓ The service should be running
        ✓ Forward lookups public servers
        ✓ Forward lookups private servers
        ✓ Reverse lookups public servers
        ✓ Reverse lookups private servers
        ✓ Alias lookups public servers
        ✓ Alias lookups private servers
        ✓ NS record lookup
        ✓ Mail server lookup

        13 tests, 0 failures

- pr002: verifying tests:

        Running test /vagrant/test/common.bats
        ✓ SELinux should be set to 'Enforcing'
        ✓ Firewall should be enabled and running
        ✓ EPEL repository should be available
        ✓ Bash-completion should have been installed
        ✓ bind-utils should have been installed
        ✓ Git should have been installed
        ✓ Nano should have been installed
        ✓ Tree should have been installed
        ✓ Vim-enhanced should have been installed
        ✓ Wget should have been installed
        ✓ Admin user piet should exist
        ✓ An SSH key should have been installed for piet
        ✓ Custom /etc/motd should have been installed

        13 tests, 0 failures
        Running test /vagrant/test/pr002/slavedns.bats
        ✓ The dig command should be installed
        ✓ The main config file should be syntactically correct
        ✓ The server should be set up as a slave
        ✓ The server should forward requests to the master server
        ✓ There should not be a forward zone file
        ✓ The service should be running
        ✓ Forward lookups public servers
        ✓ Forward lookups private servers
        ✓ Reverse lookups public servers
        ✓ Reverse lookups private servers
        ✓ Alias lookups public servers
        ✓ Alias lookups private servers
        ✓ NS record lookup
        ✓ Mail server lookup

        14 tests, 0 failures

- Nslookup: this is returning the correct IP and shows that the pr001/pr002 ns record is available



    
## Resources
/
