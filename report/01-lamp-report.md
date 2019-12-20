# Enterprise Linux Lab Report

- Student name: Piet Jacobs
- Github repo: <https://github.com/HoGentTIN/elnx-1920-sme-PietJ95.git>

Setting up basic automated installation for all upcoming servers

## Test plan

### Verify tests
- Open a terminal in the root directory of this project
- Bring up the machine with `vagrant up pu001`
- Connect to the machine with `vagrant ssh pu001`
- Run the following command and verify that all tests pass

        sudo /vagrant/test/runbats.sh
### Check functionality
- Check that surfing to https://192.0.2.10/wordpress shows the wordpress website and that it is trying to serve you a ssl certificate
- On the machine, verify that you can login to the mariadb service with

        mysql -u root -pRo0tP4sS


- If logging in works, now check that the wordpress db is present by entering:

        show databases;



## Procedure/Documentation

**Notes**  
My go to method when I don't instantly know the right way to handle a task is by trying to google my problem/requirement as closely as possible.
 
- Everything was pretty straight forward, the bertvv wordpress role was very concise and few variables needed to be set.
- I did however needed to check the test vars in the script to see kind of values it was expecting for user/pass/db name and adjusted things accordingly
- For SSL, the httpd role offers 2 variables to set your certificates on the server

### Add mariadb variables to `pu001.yml`
Creates the mariadb database and user with the neccesary rights

    mariadb_databases:
      - name: wp_db
    mariadb_users: 
      - name: piet
        password: testPass
        priv: 'wp_db.*:ALL'
    
    mariadb_root_password: 'Ro0tP4sS'

### Add Wordpress variables to `pu001.yml`
Configures wordpress with a user and links with the database created earlier

    wordpress_database: wp_db
    wordpress_user: piet
    wordpress_password: testPass

### Add httpd variables to `pu001.yml`
Enables the website to support https

    httpd_ssl_certificate_key_file: 'ca.key'
    httpd_ssl_certificate_file: 'ca.crt'

### Add rhbase variables to `pu001.yml`
This allows the http & https service to pass through the firewall

    rhbase_firewall_allow_services:
      - http
      - https

## Test report


Problems I encountered and their solution:  
- ...
    - Solution:  
    

- ...
    - Solution:   
    
## Resources

- Stackoverflow.com
- Ansible docs (by just googling my problem, leading me there)
