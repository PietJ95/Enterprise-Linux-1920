# Enterprise Linux Lab Report

- Student name: Piet Jacobs
- Github repo: <https://github.com/HoGentTIN/elnx-1920-sme-PietJ95.git>

Setting up basic automated installation for all upcoming servers

## Test plan
- Open a terminal in the root directory of this project
- Bring up the machine with `vagrant up pr011`
- Connect to the machine with `vagrant ssh pr011`
- Run the following command and verify that all tests pass

        sudo /vagrant/test/runbats.sh


- Ga naar `Deze Computer` op je host (Windows) en doe rechtermuis op een lege plaats en klik `Een netwerklocatie toevoegen`
- Vervolgens geef je `ftp://172.16.192.11/techncial` in en klik je `Volgende`
- Je kan nu een inlog definiëren, kies `alexanderd` en verbind. Het wachtwoord is ook `alexanderd`
- Dit zou een maplocatie moeten openen zonder problemen.
        
## Procedure/Documentation

Issue with vsftp remote connection using curl: 
command used for testing (from the vsftp.bats script): 

    curl ftp://172.16.192.11/public/ --user alexanderd:alexanderd
    
gives error:

    pam_unix(vsftpd:auth): authentication failure; logname= uid=0 euid=0 tty=ftp ruser=alexanderd rhost=pr011  user=alexanderd

    Oct 26 13:26:40 pr011 vsftpd[20308]: [alexanderd] FAIL LOGIN: Client "172.16.192.11"

After commenting out some lines in /etc/pam.d/vsftpd

    #%PAM-1.0
    session    optional     pam_keyinit.so    force revoke
    auth       required     pam_listfile.so item=user sense=deny file=/etc/vsftpd/ftpusers onerr=succeed
    #auth       required    pam_shells.so
    #auth       include     password-auth
    account    include      password-auth
    session    required     pam_loginuid.so
    session    include      password-auth

and retrying the curl command I get a new error:

    curl: (9) Server denied you to change to the given directory

Stuck here for now.

---
### Different approach:
Maybe something wrong with the user passwords? Ansible role bertvv.rh-base gives me a warning that passwords aren't hashed for the module to work.

* Went to https://www.mkpasswd.net/index.php and made all user passwords into crypt-sha512 hashes
* Tried using the curl command again, this time it doesn't show the error 530, but goes straight to curl error (9). Which means I did something right...   
Commenting this out was a bad idea

        #auth    include     password-auth

So I reverted this and all is well, except for the curl error (9) of course. 


The other commented out line needs to stay, implementing this with a template /etc/pam.d/vsftpd file and overwriting the one on the remote machine. (Embedded into the bertvv.vsftpd role)

---
Apparently it has to do with absolute/relative path to the server share.  
When trying to connect using following command, it works!

    curl ftp://172.16.192.11//srv/share/public/ --user anc:anc

So the relative path to the root of the ftp server is missing in /etc/vsftpd/vsftpd.conf and after some googling I found the correct way to add it: 

    local_root=/srv/shares/

Found the var in bertvv.vsftpd to manage it easiliy with ansible:

    vsftpd_local_root: /srv/shares/


More tests passing, not all of them (no_read_access still failing)

---
### ACL management
Making use of the `getfacl` command to troubleshoot the access control

**Trying it manually with**

    setfacl -m user::--- /srv/shares/management/
    setfacl -m group::--- /srv/shares/management/
    setfacl -m other:--- /srv/shares/management/
    setfacl -m g:sales:--- /srv/shares/management/
    setfacl -m g:it:--- /srv/shares/management/
    setfacl -m g:management:rwx /srv/shares/management/

This makes it work, however I'm finding difficulty using the vsftpd role for this

---
Continuation...

    vsftpd_extra_permissions:
      - folder: "/srv/shares/management"
        entity: "sales"
        etype: "group"
        permissions: "---"

      - folder: "/srv/shares/management"
        entity: "it"
        etype: "group"
        permissions: "---"
    [...]

Turns out this is the way to go, however after doing some testing I realised that the etype "other:" needs to also be set to "---" as rwx permissions; however this role throws an error when I leave out either "entity" or "etype", which makes this very very weird...  
 Will be leaving that part out for now


 **FOLLOW UP:** I actually didn't need to change "other:". I just forgot to change permissions for the "technical" group which threw errors in the test script, oops.


## Test report

    Running test /vagrant/test/pr011/samba.bats
    ✓ The ’nmblookup’ command should be installed
    ✓ The ’smbclient’ command should be installed
    ✓ The Samba service should be running
    ✓ The Samba service should be enabled at boot
    ✓ The WinBind service should be running
    ✓ The WinBind service should be enabled at boot
    ✓ The SELinux status should be ‘enforcing’
    ✓ Samba traffic should pass through the firewall
    ✓ Check existence of users
    ✓ Checks shell access of users
    ✓ Samba configuration should be syntactically correct
    ✓ NetBIOS name resolution should work
    ✓ read access for share ‘public’
    ✓ write access for share ‘public’
    ✓ read access for share ‘management’
    ✓ write access for share ‘management’
    ✓ read access for share ‘technical’
    ✓ write access for share ‘technical’
    ✓ read access for share ‘sales’
    ✓ write access for share ‘sales’
    ✓ read access for share ‘it’
    ✓ write access for share ‘it’

    22 tests, 0 failures
    Running test /vagrant/test/pr011/vsftp.bats
    ✓ VSFTPD service should be running
    ✓ VSFTPD service should be enabled at boot
    ✓ The ’curl’ command should be installed
    ✓ The SELinux status should be ‘enforcing’
    ✓ FTP traffic should pass through the firewall
    ✓ VSFTPD configuration should be syntactically correct
    ✓ Anonymous user should not be able to see shares
    ✓ read access for share ‘public’
    ✓ write access for share ‘public’
    ✓ read access for share ‘management’
    ✓ write access for share ‘management’
    ✓ read access for share ‘technical’
    ✓ write access for share ‘technical’
    ✓ read access for share ‘sales’
    ✓ write access for share ‘sales’
    ✓ read access for share ‘it’
    ✓ write access for share ‘it’

    17 tests, 0 failures

    
## Resources

- https://technicalsanctuary.wordpress.com/2012/11/01/curl-curl-9-server-denied-you-to-change-to-the-given-directory/
- https://askubuntu.com/questions/741164/how-to-change-vsftpds-default-directory-to-instead-of-the-users-home-directo
- https://www.mkpasswd.net/index.php
- https://superuser.com/questions/271034/list-samba-users
- https://askubuntu.com/questions/678934/how-to-list-all-ftp-users
- https://stackoverflow.com/questions/27646810/clone-git-using-ftp-server-denied-you-to-change-to-the-given-directory
- https://unix.stackexchange.com/questions/109311/new-local-user-cant-login-to-vsftpd
- https://bbs.archlinux.org/viewtopic.php?id=72117
- https://linuxconfig.org/how-to-manage-acls-on-linux
