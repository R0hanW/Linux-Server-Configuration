# Linux Server Configuration
This is a linux server that hosts this [Catalog App](https://github.com/R0hanW/catalog-app) at the url [52.14.119.10]()

## Prerequisites
To connect to the server, [git](https://git-scm.com/downloads) is needed.

## Quickstart
1. Download the private key in the github repo,
2. Connect to the server by typing ```ssh grader@52.14.119.10  -i privateKey.pem -p 2200``` into the terminal.

## Walkthrough
### Changing the SSH Port
1. Use the command ```sudo nano /etc/ssh/sshd_config``` and edit the port that ssh listens for to 2200.
2. Restart ssh using the command ```sudo service ssh restart```. You should get disconnected from the server.

### Configuring the Firewall
1. Go to the networking tab on Amazon Lightsail.
2. Configure the firewall to allow http at port 80, udp at port 123, and tcp at port 2200.
3. Configure the uncomplicated firewall (UFW) to block the same ports:
 ```sudo ufw default deny incoming```
 ```sudo ufw default allow outgoing```
 ```sudo ufw allow 2200/tcp```
 ```sudo ufw allow 80/tcp```
 ```sudo ufw allow 123/udp```
 ```sudo ufw enable```

### Creating a New User
1. ```sudo adduser grader```
2. Give the user access to sudo ```usermod -aG sudo username```
3. Allow user to use sudo without typing in the password:
   ```sudo visudo```
   Type ```$USER ALL=(ALL) NOPASSWD: ALL``` at the bottom of the file.

### Allow the New User to SSH 
1. Login into the account ```su -grader```
2. Make an ssh directory ```mkdir .ssh```
3. Touch ```.ssh/authorized_keys```
4. Log out of the account ```ctrl+d``` and copy the key in the file ```.ssh/authorized_keys```.
5. Log back in and paste the key into the file.
6. Change the file permissions
 ```chmod 700 .ssh```
 ```chmod 600 .ssh/authorized_keys```
7. Reload ssh ```sudo service ssh restart```

### Change Timezone
1. ```sudo timedatectl set-timezone UTC```

### Download Catalog App
1. Go to the write directory ```cd /var/www```
2. Create an application directgory 
    ```mkdir catalogApp```
    ```cd catalogApp```
3. Download the app ```git clone https://github.com/R0hanW/catalog-app```
4. Rename the directory ```sudo mv catalog-app flaskApp```
5. Rename application.py to __init__.py 
   ```cd flaskApp```
    ```sudo mv application.py __init.py__```
6. Download all necessary programs for running your application (Flask, SQLAlchemy, requests, etc.)

### Set Up postgreSQL
1. ```sudo apt-get install postgresql postgresql-contrib```
2. Create a postgresql user ```sudo -u postgres createuser psql -d -P```
3. Add a record for the new user:
    a. Open the file ```sudo nano /etc/postgresql/9.3/main/pg_hba.conf```
   b.  Add information to the table
     ```
    # TYPE  DATABASE        USER            ADDRESS                 METHOD
    local   all             psql                                    md5
    ```
    c. restart postgresql```sudo service postgresql restart```
4. Create a database ```createdb -U psql catalog -T template0```
5. Configure catalog app code so that the postgresql database is used ```engine = create_engine('postgresql://psql:(password)@localhost/catalog')```

### Set Up Apache 
1. Install apache2 ```sudo apt-get install apache2```
2. Make a configuration file for your app:
    a. Create the file ```sudo nano /etc/apache2/sites-available/flaskApp.conf```
    b. Input the code into the file
        ```
        <VirtualHost *:80>
            ServerName 52.14.119.10
            ServerAdmin rohan.avatar@gmail.com
            WSGIScriptAlias / /var/www/catalogApp/flaskApp.wsgi
        <Directory /var/www/catalogApp/flaskApp/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/catalogApp/flaskApp/static
        <Directory /var/www/catalogApp/flaskApp/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
        ```
3. Restart apache2 ``` sudo service apache2 restart```

### Set Up WSGI
1. Download wsgi ```sudo apt-get install libapache2-mod-wsgi python-dev```
2. Start wsgi ```sudo apt-get install libapache2-mod-wsgi python-dev```
3. Create a wsgi file for your application
    a. Create the file ```sudo nano /var/www/catalogApp/flaskApp.wsgi```
    b. Input the code into the file 
    ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/catalogApp/")

    from flaskApp import app as application
    application.secret_key = 'Add your secret key'
    ```
4. Restart apache ```sudo service apache2 restart```
5. The website is now running! Visit [52.14.119.10](http://52.14.119.10) to view it.

## Built with
Amazon lightsail was used to host the application. Apache2, wsgi, Flask, postgreSQL, SQLAlchemy, requests, and httplib2 were install in order for the application to run. Some packages, like Flask, installed additional packages in order to run.

## References
[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) 
[Linuxize](https://linuxize.com/post/how-to-create-a-sudo-user-on-ubuntu/)



