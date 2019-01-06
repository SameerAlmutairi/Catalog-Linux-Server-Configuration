# FSND Project (3) - Linux Server Configuration 

By Sameer Almutairi

### Project overview
> You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

### Server Details
|       Name        |      Value        |
| :---------------: |:-----------------:|
| IP Address        | **3.120.41.202**  |
| PORT              | **2200**          |
| USERNAME          | **grader**        |
| SITE URL          | **http://3.120.41.202.xip.io**        |

## Configuration Steps
### 1. Create Ubuntu Server Instance on Amazon Lightsail 
* Login into [Amazon Lightsail](https://lightsail.aws.amazon.com) or create new account.
* click `Create instance`
* Choose `Linux/Unix platform`, `OS Only` and `Ubuntu 16.04 LTS`
* Choose a instance plan eg. $3.5/month (Free for the First Month).
* Click the Create button to create the instance.

### 2. Setup SSH Connection to The Server
* from [Account Page](https://lightsail.aws.amazon.com/ls/webapp/account/keys) , click on `SSH KEYS` and download the default private key.
* Move the downloaded private key `LightsailDefaultKey-eu-central-1.pem` into `~/.ssh` the rename it to `ubuntu-lightsail.rsa`
* in terminal inside `.ssh` folder, change the file permission using the next command:
    ```console
    YourMachineName:.ssh $ chmod 600 ubuntu-lightsail.rsa
    ```
* then connnect to the server using this command
     ```console
    YourMachineName:.ssh $ ssh -i ~/.ssh/ubuntu-lightsail.rsa ubuntu@18.194.38.119
    ```

### 3. Update and Upgrade the server
```console
    YourMachineName:.ssh $ ssh -i ~/.ssh/ubuntu-lightsail.rsa ubuntu@18.194.38.119
```
```console
    ubuntu@YourServer:~$ sudo apt-get update
 ```
```console
    ubuntu@YourServer:~$ sudo apt-get upgrade or sudo apt-get dist-upgrade
```

### 4. Change the SSH port from 22 to 2200
```console
    ubuntu@YourServer:~$ sudo nano /etc/ssh/sshd_config
```
> then change the `Port 22` to `Port 2200`

### 5. Configure the Uncomplicated Firewall (UFW)
* follow these instructions:
```console 
    ubuntu@YourServer:~$ sudo ufw status
    Status: inactive
```
```console
    ubuntu@YourServer:~$ sudo ufw default deny incoming
    Default incoming policy changed to 'deny'
    (be sure to update your rules accordingly)
```
```console
    ubuntu@YourServer:~$ sudo ufw default allow outgoing
    Default outgoing policy changed to 'allow'
    (be sure to update your rules accordingly)
```
```console 
    ubuntu@YourServer:~$ sudo ufw allow ssh
    Rules updated
    Rules updated (v6)
```
```console 
    ubuntu@YourServer:~$ sudo ufw allow 2200/tcp
    Rules updated
    Rules updated (v6)
```
```console 
    ubuntu@YourServer:~$ sudo ufw allow www
    Rules updated
    Rules updated (v6)
```
```console 
    ubuntu@YourServer:~$ sudo ufw allow 123/udp
    Rules updated
    Rules updated (v6)
```
```console 
    ubuntu@YourServer:~$ sudo ufw deny 22
    Rules updated
    Rules updated (v6)
```
```console 
    ubuntu@YourServer:~$ sudo ufw enable
   Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
    Firewall is active and enabled on system startup
```
```console 
    ubuntu@YourServer:~$ sudo ufw status
    Status: active
    To                         Action      From
    --                         ------      ----
    22                         DENY        Anywhere                  
    2200/tcp                   ALLOW       Anywhere                  
    80/tcp                     ALLOW       Anywhere                  
    123/udp                    ALLOW       Anywhere                  
    22 (v6)                    DENY        Anywhere (v6)             
    2200/tcp (v6)              ALLOW       Anywhere (v6)             
    80/tcp (v6)                ALLOW       Anywhere (v6)             
    123/udp (v6)               ALLOW       Anywhere (v6)
```
* go to  `Network` tab in created lightsail instance and click on `Edit rules` in `Firewall` and allow those ports:

    |       Application        |      Protocol        |      Port range       |
    | :----------------------: |:--------------------:|:---------------------:|
    |           HTTP           |        TCP           |         80            |
    |           Custom         |        UDP           |         123           |
    |           Custom         |        TCP           |         2200          |

### 6. Create user `grader`
```console 
    ubuntu@YourServer:~$ sudo adduser grader
```
> Enter the user password (twice) and fill out this new user information

### 7. Give `grader` sudo permission
* create a new file in sudoers directory:
```console 
    ubuntu@YourServer:~$ sudo nano /etc/sudoers.d/grader
```
* add this text ``grader ALL=(ALL:ALL) ALL`` then save

### 8. Create SSH key for `grader` using ssh-keygen tool
* On your local terminal, follow the instructions to create SSH key:
* run ssh-keygen
        ```console
        YourMachineName:~$ ssh-keygen
        ```
* Enter file name (mine is `grader_rsa`) in your local directory `~/.ssh`
* Two files will be generated ( `~/.ssh/grader_rsa` and `~/.ssh/grader_rsa.pub`)
* Run `nano ~/.ssh/grader_rsa.pub` and copy file contents
* Login to the server using `grader`
* Create `.ssh` folder then `sudo nano .ssh/authorized_keys`
* Paste created public key `grader_rsa.pub` into the `authorized_keys` file
* change .ssh directory permission
     ```console
        grader@YourServer:~$ sudo chmod 700 ~/.ssh
    ```
* change `grader_rsa.pub` permission
    ```console
        grader@YourServer:~$ sudo chmod 644 ~/.ssh/authorized_key
    ```
* Now, You can login using 'grader' private key, using
    ```console
        YourMachineName:~$ ssh -i ~/.ssh/grader_key -p 2200 grader@18.194.38.119
    ```

### 9. Configure the local timezone to UTC
* While logged in as 'grader', configure UTC time zone:
    ```console
        grader@YourServer:~$ sudo dpkg-reconfigure tzdata
    ```

### 10. Install and configure Apache to serve a Python mod_wsgi application
* As logged in as `grader`, install Apache:
    ```console
        grader@YourServer:~$ sudo apt-get install apache2
        grader@YourServer:~$ sudo apt-get install python-setuptools libapache2-mod-wsgi
    ```    

### 11. Install and configure PostgreSQL
* As logged in as `grader`, install PostgreSQL:
    ```console
        grader@YourServer:~$ sudo apt-get install postgresql postgresql-contrib
    ```
* to disable remote connections with PostgreSQL, open the file:
    ```console
        grader@YourServer:~$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf
    ```
* you should see:

| TYPE  | DATABASE |  USER   | ADRESS  |   METHOD |
| :-------------: |:-------------:|:-------------:|:-------------:|:-------------:|
|    local  |        all           |         postgres            |                      |         peer      |
|   local  |        all           |         all           |                     |         peer   |
|  host    |        all           |         all          |         127.0.0.1/32          |         md5      |
|  host    |        all           |         all          |         ::1/128          |         md5      |

* Now, switch to `postgres` user:
    ```console
        grader@YourServer:~$ sudo su - postgres
    ```
* Open PostgresSQL terminal:
    ```console
        postgres@YourServer:~$ psql
    ```
* Create `catalog` user with password then give it `create database` role:
    ```console
        postgres=# create user catalog with password 'catalog';
        CREATE ROLE
        postgres=# alter role catalog CREATEDB;
        ALTER ROLE
    ```
* Create `catalog` Database:
    ```console
        postgres=# Create Database catalog;
        CREATE DATABASE
    ```
* Give `catalog` user permission to 'catalog' Database:
    ```console
        postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
        GRANT
    ```
* to exit: `\q` then logout postgres user: `exit`

### 12. Install Git and Setup Catalog application
* Install Git:
    ```console
        grader@YourServer:~$ sudo apt-get install git
    ```
* go to 'www' directory:
    ```console
        grader@YourServer:~$ cd /var/www
    ```
* then, create application directory and move to it:
    ```console
        grader@YourServer:/var/www$ sudo mkdir Apps
        grader@YourServer:/var/www$ cd Apps
    ```
* Clone catalog app rename the folder then move to it:
    ```console
        grader@YourServer:/var/www$ sudo git clone https://github.com/SameerAlmutairi/Catalog-Project.git
        grader@YourServer:/var/www$ sudo mv Catalog-Project CatalogProject 
        grader@YourServer:/var/www$ cd CatalogProject/
    ```
* In `app.py`, replace:
    ```python   
        # app.debug = True 
        #  app.run(host='0.0.0.0', port=5000)`
        app.run()
    ```
* and replace':
    ```python  
    #engine = create_engine('sqlite:///catalog.db?check_same_thread=False')
    engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
    ```
* Install the requirements:
    * Pip use: `sudo apt-get install python-pip`
    * Flask FrameWork use: `pip install flask`
    * SQLAlchemy use: `pip install SQLAlchemy`
    * Oauth2client use: `pip install oauth2client`
    * Psycopg2 use: `sudo apt-get install python-psycopg2`

* Setup app database using then Insert default database data:
 ```console
        grader@YourServer:/var/www/Apps/CatalogProject$ sudo python catalog_DB.py
        grader@YourServer:/var/www/Apps/CatalogProject$ sudo python catalog_DB_Data.py
```

### 13. Configure and Enable a New Virtual Host
* Create app.conf to edit:
```console
    grader@YourServer:/var/www/Apps/CatalogProject$ sudo nano /etc/apache2/sites-available/catalog.conf
```
* Add the following lines of code to the file to configure the virtual host:
```
    <VirtualHost *:80>
        ServerName 3.120.41.202
        ServerAdmin grader@3.120.41.202
        WSGIScriptAlias / /var/www/Apps/catalogproject/catalogApp.wsgi
        WSGIApplicationGroup %{GLOBAL}
        <Directory /var/www/Apps/catalogproject/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/Apps/catalogproject/static
        <Directory /var/www/Apps/catalogproject/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
```
* Enable the virtual host: `sudo a2ensite App.conf`
* Disable the default virtual host: `sudo a2dissite 000-default.conf`
* Create the .wsgi File under /var/www/Apps/catalogproject:
```console
    grader@YourServer:/var/www/Apps/catalogproject$ sudo nano catalogApp.wsgi
```
* Add the following  code to the `catalogApp.wsgi` file:
```python
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/Apps/catalogproject/")

    from app import app as application
    application.secret_key = 'Add your secret key'
```
* Restart Apache using `sudo service apache2 restart`

### 14. Update Google Credentials 
* To get the Google login working there are a few additional steps:
    * Go to Google Dev Console and Login 
    * Go to Credentials > OAuth consent screen > Authorised domains and add `xip.io` 
    * Go to Credentials >  select OAuth 2.0 client ID > add `http://3.120.41.202.xip.io` in `Authorised JavaScript origins` and add `http://3.120.41.202.xip.io/gconnect` in `Authorised redirect URIs`
    * then select save and download Client_secret.json	.
    * Copy Client_secret.json data and paste the in the Client_secret.json in your server.
    * then restart the server `sudo apache2ctl restart` and `sudo service apache2 reload`
    * then goto `http://3.120.41.202.xip.io/login` to login using google authentication.
