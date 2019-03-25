# udacity-linux-server-configuration



IP address:  52.59.3.233

SSH port:  2200.

Application URL:  http://52.59.3.233.xip.io/

host provider :  amazon lightsail  

OS : ubuntu 18.04

# Configuration steps
##1. Update all currently installed packages :
  -  `$ sudo apt-get update`
  -  `$ sudo apt-get upgrade`
  
##2. Create new user :
  - Run `$ sudo adduser grader` to create a new user named grader
  - add grader to sudo group 
  `$ usermod -aG sudo username`
  - then switch to grader user using
  `$ sudo su - grader`
  
##3.Configure Firewall : 
3. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  - `sudo ufw default deny incoming`
  - `sudo ufw default allow outgoing `
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow http`
  - `sudo ufw allow ntp`
  - `sudo ufw enable`

##4. set login restrictions : 
- Change SSH port from 22 to 2200
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change the port from 22 to 2200
  - Change `PermitRootLogin without-password` line to  `PermitRootLogin no`
  - Change `PasswordAuthentication ` to `no`
  - Restart ssh with `sudo service ssh restart`
  

  
5. Configure the local timezone to UTC
  - Run `$ sudo dpkg-reconfigure tzdata` and then choose UTC
 
##5. Configure key-based authentication for grader user :
### Step 1 — Create the RSA Key Pair :
- on your local machine , use this command 
`       $ssh-keygen
`
After entering the command, you should see the following output:

    ```
    Generating public/private rsa key pair.
    Enter file in which to save the key (/your_home/.ssh/grader):
    ```

Press  `ENTER`  to save the key pair into the  `.ssh/`  subdirectory in your home directory, or specify an alternate path.
###Step 2 — Copy the Public Key to Ubuntu Server
To display the content of your  `grader.pub`  key, type this into your local computer:

```
cat ~/.ssh/grader.pub

```

You will see the key's content, which should look something like this:

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCqql6MzstZYh1TmWWv11q5O3pISj2ZFl9HgH1JLknLLx44+tXfJ7mIrKNxOOwxIxvcBF8PXSYvobFYEZjGIVCEAjrUzLiIxbyCoxVyle7Q+bqgZ8SeeM8wzytsY+dVGcBxF6N4JS+zVk5eMcV385gG3Y6ON3EG112n6d+SMXY0OEBIcO6x+PnUSGHrSgpBgX7Ks1r7xqFa7heJLLt2wWwkARptX7udSq05paBhcpB0pHtA1Rfz3K2B+ZVIpSDfki9UVKzT8JUmwW6NNzSgxUfQHGwnW7kj4jp4AT0VZk3ADw497M2G/12N0PPB5CnhHf7ovgy6nL1ikrygTKRFmNZISvAcywB9GVqNAVE+ZHDSCuURNsAInVzgYo9xgJDW8wUw2o8U77+xiFxgI5QSZX3Iq7YLMgeksaO4rBJEa54k8m5wEiEE1nUhLuJ0X/vh2xPff6SQ1BL/zkOhvJCACK6Vb15mDOeCSq54Cr7kvS46itMosi/uS66+PujOO+xt/2FWYepz6ZlN70bRly57Q06J+ZJoc9FfBCbCyYH7U/ASsmY095ywPsBo1XQ9PqhnN1/YOorJ068foQDNVpm146mUpILVxmq41Cj55YKHEazXGsdBIbXWhcrRf4G2fJLRcGUr9q8/lERo9oxRm5JFX6TCmj6kmiFqv+Ow9gI0x8GvaQ== demo@test

```
  - Create a **.ssh** directory in the **grader** home directory:
  `$ mkdir .ssh`
  `$ chmod 700 .ssh`
  `$ touch .ssh/authorized_keys`
  `$ chmod 600 .ssh/authorized_keys` 
  `$cat >> .ssh/authorized_keys`
Paste the public key into the **.ssh/authorized_keys** file

###Step 3 — Authenticate to Ubuntu Server Using SSH Keys
- login to your server using ssh 

`ssh -i ~/.ssh/grader grader@52.59.3.233 
`

##6. Install Apache
  - `sudo apt install apache2`

##7. Install mod_wsgi
  - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  - Enable mod_wsgi with `sudo a2enmod wsgi`
  - Start the web server with `sudo service apache2 start`

  
##8. Clone the Catalog app from Github
  - Install git using: `sudo apt-get install git`
  - `cd /var/www`

  - Clone your project from github 
```shell
      git clone https://github.com/abousamraa/item-catalog-project.git flaskapp
```
  - Create a flaskapp.wsgi file in `/var/www`, then add this inside:
  ```
#!/usr/bin/python3
import sys
sys.path.insert(0, '/var/www/flaskapp/')
sys.stdout = sys.stderr
from runserver import app as application
  ```
  
  - create an empty file in flaskapp directory named  `__init__.py `
  

##9. Install Flask and other requirements
  - Install pip with `sudo apt-get install python3-pip`
  - Install requirements from requirements.txt
  `sudo pip3 install -r requirements.txt`

  
##10. Configure apache2 
  - Run this: `sudo nano /etc/apache2/sites-available/flaskapp.conf`
  - Paste this code: 
  
```shell
  <VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
        WSGIDaemonProcess flaskapp  threads=5
        WSGIScriptAlias / /var/www/flaskapp.wsgi
        <Directory /var/www/flaskapp>
            WSGIProcessGroup flaskapp
            WSGIApplicationGroup %{GLOBAL}
            Require all granted
        </Directory>

        Alias /static /var/www/flaskapp/project2/static/
        <Directory /var/www/flaskApp/project2/static/>
                Require all granted
        </Directory>
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

  
```
  - Enable the virtual host `sudo a2ensite flaskapp`
  - disable the default host `sudo a2dissite 000-default`
  - To start the web server when it is stopped, type:

`sudo systemctl start apache2
`
- To stop and then start the service again, type:

`sudo systemctl restart apache2
`
If you are simply making configuration changes, Apache can often reload without dropping connections. To do this, use this command:

`sudo systemctl reload apache2
`
##11. Install and configure PostgreSQL
  - `sudo apt install postgresql postgresql-contrib`
  - change to postgres user `sudo su - postgres`
  - run `psql`
  - `CREATE USER grader WITH PASSWORD 'grader';`
  - `ALTER USER grader CREATEDB;`
  - `CREATE DATABASE catalog WITH OWNER grader;`
  - `\c catalog`
  - `REVOKE ALL ON SCHEMA public FROM public;`
  - `GRANT ALL ON SCHEMA public TO grader;`
  - `\q`
  - `exit`
  - Change  line in  `config.py`  to: 
  `SQLALCHEMY_DATABASE_URI = 'postgresql://grader:grader@localhost/catalog'`
  - populate data base
  `python /var/www/catalog/catalog/db_populate.py`
  - By default no remote connections to the database are allowed. 
  
##12. switch time to UTC 

  - To switch to UTC, simply execute `sudo dpkg-reconfigure tzdata`, scroll to the bottom of the Continents list and select Etc or None of the above; in the second list, select UTC
  
##13. Run the server 
  - before  `sudo service apache2 restart`
  - go to `http://52.59.3.233.xip.io/` to see the application running 
  
##14.Debugging

If you are getting any error, check out Apache's error log for debugging:

`$ sudo cat /var/log/apache2/error.log`
##15.References
1. https://aws.amazon.com/premiumsupport/knowledge-center/new-user-accounts-linux-instance/
1. https://linuxize.com/post/how-to-create-a-sudo-user-on-ubuntu/
1. https://www.cyberciti.biz/faq/ubuntu-18-04-update-installed-packages-for-security/
1. https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04
1. https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-18-04
1. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
1. http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/
1. https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps
1. https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04
1. https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt
