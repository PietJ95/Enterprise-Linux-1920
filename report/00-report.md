# Enterprise Linux Lab Report

- Student name: Piet Jacobs
- Github repo: <https://github.com/HoGentTIN/elnx-1920-sme-PietJ95.git>

Setting up basic automated installation for all upcoming servers

## Test plan

- All tests need to pass
- Verify that 'ssh piet@192.0.2.50' doesn't require a password and connects you to the vm
- Verify that after signing into the vm you see the ascii from the motd banner

## Procedure/Documentation

My go to method when I don't instantly know the right way to handle a task is by trying to google my problem/requirement as closely as possible.
 
- Most requirements I handled with ease:
    - Custom /etc/motd: found a nice ansible ascii file for the banner and easily added the link to the file in ansible code
    - Software installations all were very straight forward with yum

## Test report

All installation requirements passed instantly

Problems I encountered and their solution:  
- Creating an "admin" user
    - Solution:  
    I did a lot of searching about what user group lies closest to the admin role andactually gives the sudo permission.  
    Finally figured out it was the role "wheels" and not "adm" or "root"

- I had issues with properly making a public ssh key (tried using putty but ansible kept giving me wrong key format error)
    - Solution:   
    Turns out I already had a id_rsa.pub in /.ssh on my host machine (it's there for github ssh), so I also just used that one for my vm's.
## Resources

- Stackoverflow.com
- Ansible docs (by just googling my problem, leading me there)
