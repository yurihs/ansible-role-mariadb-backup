# Ansible Role: MariaDB Backups

- Installs a script to back up MariaDB databases.
- Supports backing up multiple hosts.
- Manages cron entries for regular execution.


## What is backed up?

- All readable non-system databases, to separate files. Command used: `mysqldump --single-transaction --quick --lock-tables=false $database`


## Role variables (default values)

~~~yaml
mariadb_backup_list:
  - name: default
    db_username: root
    db_hostname: localhost
    db_port: 3306
    cron:
      minute: '0'
      hour: '*'
      day: '*'
      month: '*'
      weekday: '*'
~~~

Which MariaDB instances to backup (connection info), and (optionally) how
often.

These entries are stored in configuration files, so you must consider the
security implications of storing passwords in plaintext (when you specify them
using `db_password`, see examples below). By default, these files will be
readable only by their owner (usually, `root`).

When removing an item from the list, remember to change to use the `state`
parameter to remove the configuration file and the cron job from the system
(see examples below).

~~~yaml
mariadb_backup_config_dir: /etc/mariadb_backup
~~~

Where to store the configuration.

~~~yaml
mariadb_backup_default_output_dir: /srv/mariadb_backup
~~~

Where to store the backups. Each entry will have its own directory inside
here. This variable may be overridden by each entry in the backup list.

~~~yaml
mariadb_backup_default_date_format: "%Y-%m-%d_%H-%M"
~~~

How to format the output directory for each backup. May be overriden by each
entry in the backup list.

## Examples

### Hourly backup of a local MariaDB server

~~~yaml
mariadb_backup_list:
  - name: default
    db_username: root
    db_hostname: localhost
    db_port: 5432
    cron:
      minute: '0'
      hour: '*'
      day: '*'
      month: '*'
      weekday: '*'
~~~

This configuration will result in the following structure:

~~~
/srv/
  mariadb_backup/
    default/
      2019-01-01_00-00/
        database-a.sql.gz
        database-b.sql.gz
      2019-01-01_01-00/
        database-a.sql.gz
        database-b.sql.gz
~~~

### No automatic backups, just install the script and configuration

~~~yaml
mariadb_backup_list:
  - name: default
    db_username: root
    db_hostname: localhost
    db_port: 5432
~~~

### Daily backup of a remote server, using a password, and output to a different directory

~~~yaml
mariadb_backup_list:
  - name: production
    db_username: backup
    db_password: "hunter2"
    db_hostname: db.example.com
    db_port: 5432
    output_dir: /opt/prod_db_bak
    cron:
      minute: '0'
      hour: '0'
      day: '*'
      month: '*'
      weekday: '*'
~~~

This configuration will result in the following structure:

~~~
/opt/
  prod_db_bak/
    2019-01-01_00-00/
      users.sql.gz
      posts.sql.gz
    2019-01-02_00-00/
      users.sql.gz
      posts.sql.gz
~~~

### Remove a previously defined configuration

~~~yaml
mariadb_backup_list:
  - name: production
    state: absent
~~~
