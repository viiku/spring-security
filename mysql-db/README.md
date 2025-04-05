# Installing MySQL on ubuntu

## Logging to MYSQL
1. Check Root Authentication Method
Run the following command to check how root is authenticated:

```
sudo mysql -u root
SELECT user, host, plugin FROM mysql.user WHERE user='root';
```
If the output shows auth_socket like this:
+------+-----------+-------------+
| user | host     | plugin      |
+------+-----------+-------------+
| root | localhost | auth_socket |
+------+-----------+-------------+

It means root can only log in via sudo mysql, not with a password.

## Creating User
CREATE USER 'vikku'@'%' IDENTIFIED BY 'vikku';
GRANT ALL PRIVILEGES ON *.* TO 'vikku'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;

### create user with
Username: workbench_user
Password: YourStrongPassword
Hostname: Use 127.0.0.1 or your server's IP
Port: 3306

### Login to MYSQL
udo mysql -u vikku -p
