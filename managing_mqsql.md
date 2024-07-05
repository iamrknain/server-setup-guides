# Table of Contents:
1. [Access Root User Without Password](#access-root-user-without-password)
2. [Some Common Errors During Safe Mode MySql](#some-common-errors-during-safe-mode-mysql)
3. [Change Username and Password](#change-username-and-password)
4. [Change Database Name Using MySql Dump](#change-database-name-using-mysql-dump)
5. [Connect To Remote MySql Server](#connect-to-remote-mysql-server)

## Access Root User Without Password
```bash
# stop mysql server 

sudo systemctl stop mysql # or forcefully stop: sudo systemctl stop mysql.service
sudo mysqld_safe --skip-grant-tables &  # Start MySQL in safe mode, skipping the grant tables that store the passwords.
sudo service mysql start # Start server and connect to MySql server
sudo mysql -u root
```
```sql
use mysql;
show tables;
describe user;
UPDATE user SET authentication_string=PASSWORD('new_password') WHERE User = 'root'; -- Set new Root password
FLUSH PRIVILEGES;
exit;
```
```bash
sudo systemctl restart mysql
tail -n 50 /var/log/mysql/error.log # Check for error logs
```

## Some Common Errors During Safe Mode MySql
### If Access Denied Error:
- Ensure that the root user has the necessary privileges to connect from localhost (`'root'@'localhost'`). Grant all privileges as:
```sql
-- login as root user

GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### If Missing Directory Error (socket file DNE)
```bash 
sudo mkdir -p /var/run/mysqld # Create Missing Directory
sudo chown mysql:mysql /var/run/mysqld # Ensure that the directory is owned by the MySQL user and group
```

### If Alreddy Running Process Error:
```bash
# Stop servers

sudo mysqladmin shutdown  # or use: sudo systemctl stop mysqld/mysql/mariadb

# Kill MySql Process if above don't work 

ps aux | grep mysqld #  Check Running MySQL Processes
sudo pkill -f process_name #  sudo pkill -f mysqld_safe , sudo pkill -f mysqld
```

## Change Username and Password
```bash
mysql -u root -p #  Login as root
```
```sql
RENAME USER 'old_user'@'localhost' TO 'new_user'@'localhost'; -- Change username
ALTER USER 'new_user'@'localhost' IDENTIFIED BY 'new_password'; -- Change password
FLUSH PRIVILEGES;
```
```bash
# verify

mysql -u new_user -p # and enter password.
```


## Change Database Name Using MySql Dump
- First dump data to backup*.mysql file and then write data to new database.

```bash
# Bash

mysqldump -u root -p old_db > old_db_backup.sql # dump
mysql -u root -p -e "CREATE DATABASE new_db;"   # create new db
mysql -u root -p new_db < old_db_backup.sql # Import to new db

# Grant permissions as per required

mysql -u root -p # Login
CREATE USER 'new_db_user'@'localhost' IDENTIFIED BY 'new_db_user_password';
GRANT ALL ON *.* TO 'new_db_user'@'localhost';

FLUSH PRIVILEGES;

SELECT * FROM db WHERE Db = 'new_db';` # verify

DROP DATABASE old_db; # drop any unnecessary db
```


## Connect To Remote MySql Server
```bash
# Connect to remote machine/vm and check status

sudo systemctl status mysql

# To allow access from all ports :

sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf # Open config file
# and change bind-address to 0.0.0.0

# Log in to MySQL Server as root user and create users with the appropriate permissions :

mysql -u root -p # and enter root password
```

```sql
-- Allow access from specific remote IP address

CREATE USER 'specific_user'@'remote_ip' IDENTIFIED BY 'specific_user_password';
GRANT ALL PRIVILEGES ON this_database.* TO 'specific_user'@'remote_ip';

-- Allow access from any remote host

CREATE USER 'specific_user'@'%' IDENTIFIED BY 'specific_user_password';
GRANT ALL PRIVILEGES ON this_database.* TO 'specific_user'@'%';

-- Allow access from localhost

CREATE USER 'specific_user'@'localhost' IDENTIFIED BY 'specific_user_password';
GRANT ALL PRIVILEGES ON this_database.* TO 'specific_user'@'localhost';

FLUSH PRIVILEGES;
exit;
```

```bash
# verify 
sudo systemctl restart mysql

# Test the connection using the MySQL client:

mysql -u specific_user -p -h public_ip # Remotehost
mysql -u specific_user -p -h 127.0.0.1 # Localhost

```
