# Connexion en root impossible après installation

A partir de Ubuntu 18.04 et MySQL 5.7, le mot de passe de root n'est pas demandé lors de l'installation.

Voilà une méthode pour le définir :

```bash
sudo service mysql stop
sudo mkdir -p /var/run/mysqld
sudo chown mysql:mysql /var/run/mysqld
sudo /usr/sbin/mysqld --skip-grant-tables --skip-networking &
mysql -u root
```

Une fois connecté à MySQL en CLI exécutez ces requêtes SQL :
```sql
FLUSH PRIVILEGES;
USE mysql;
UPDATE user SET authentication_string=PASSWORD("root") WHERE User='root';
UPDATE user SET plugin="mysql_native_password" WHERE User='root';
exit;
```

Vous pouvez maintenant vous connecter à MySQL via `root` / `root`.

# Sources

[linuxconfig.org](https://linuxconfig.org/how-to-reset-root-mysql-password-on-ubuntu-18-04-bionic-beaver-linux)
