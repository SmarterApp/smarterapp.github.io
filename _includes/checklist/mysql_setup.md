## Create AWS MySQL Instance
* Create server instance to host the MySQL instance that will support ProgMan
* Create or choose an AWS security group with the following ports for inbount TCP traffic (can be done during instance creation):
  * 22
  * 3306
* Remove `apparmor`:
  * `sudo /etc/init.d/apparmor stop`
  * `sudo update-rc.d -f apparmor remove`
  * `sudo apt-get --purge remove -y apparmor apparmor-utils libapparmor-perl libapparmor1`
* Update package manager:
  * `sudo apt-get update`
  * `sudo apt-get upgrade -y`
* Install MySQL and package dependencies:
  * `sudo apt-get install -y ntp php5-mysql mysql-server-5.6 python-mysqldb libapache2-mod-auth-mysql`
* Create directories:
  * `sudo mkdir -p /var/tmp/mysql`
  * `sudo chown mysql:mysql /var/tmp/mysql`
* Update `/etc/mysql/my.cnf` to appear as follows:

~~~~
#
# The MySQL database server configuration file.
#
# You can copy this to one of:
# - "/etc/mysql/my.cnf" to set global options,
# - "~/.my.cnf" to set user-specific options.
#
# One can use all long options that the program supports.
# Run program with --help to get a list of available options and with
# --print-defaults to see which it would actually understand and use.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

# This will be passed to all mysql clients
# It has been reported that passwords should be enclosed with ticks/quotes
# escpecially if they contain "#" chars...
# Remember to edit /etc/mysql/debian.cnf when changing the socket location.
[client]
port        = 3306
socket      = /var/run/mysqld/mysqld.sock

# Here is entries for some specific programs
# The following values assume you have at least 32M ram

# This was formally known as [safe_mysqld]. Both versions are currently parsed.
[mysqld_safe]
socket      = /var/run/mysqld/mysqld.sock
nice        = 0

[mysqld]
#
# * Basic Settings
#
user        = mysql
pid-file    = /var/run/mysqld/mysqld.pid
socket      = /var/run/mysqld/mysqld.sock
port        = 3306
#basedir        = /opt/mysql/server-5.6
basedir     = /usr
datadir     = /var/lib/mysql
tmpdir      = /var/tmp/mysql
#lc-messages-dir    = /opt/mysql/server-5.6/share
lc-messages-dir = /usr/share/mysql/english
skip-external-locking
#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
#bind-address       = 127.0.0.1
#
# * Fine Tuning
#
key_buffer      = 16M
max_allowed_packet  = 16M
thread_stack        = 192K
thread_cache_size       = 8
# This replaces the startup script and checks MyISAM tables if needed
# the first time they are touched
myisam-recover         = BACKUP
#max_connections        = 100
#table_cache            = 64
#thread_concurrency     = 10
#
# * Query Cache Configuration
#
query_cache_limit   = 1M
query_cache_size        = 16M
#
# * Logging and Replication
#
# Both location gets rotated by the cronjob.
# Be aware that this log type is a performance killer.
# As of 5.1 you can enable the log at runtime!
#general_log_file        = /var/log/mysql/mysql.log
#general_log             = 1
#
# Error log - should be very few entries.
#
log_error = /var/log/mysql/error.log
#
# Here you can see queries with especially long duration
#log_slow_queries   = /var/log/mysql/mysql-slow.log
#long_query_time = 2
#log-queries-not-using-indexes
#
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
#server-id      = 1
#log_bin            = /var/log/mysql/mysql-bin.log
expire_logs_days    = 10
max_binlog_size         = 100M
#binlog_do_db       = include_database_name
#binlog_ignore_db   = include_database_name
#
# * InnoDB
#
# InnoDB is enabled by default with a 10MB datafile in /var/lib/mysql/.
# Read the manual for more InnoDB related options. There are many!
#
# * Security Features
#
# Read the manual, too, if you want chroot!
# chroot = /var/lib/mysql/
#
# For generating SSL certificates I recommend the OpenSSL GUI "tinyca".
#
# ssl-ca=/etc/mysql/cacert.pem
# ssl-cert=/etc/mysql/server-cert.pem
# ssl-key=/etc/mysql/server-key.pem



[mysqldump]
quick
quote-names
max_allowed_packet  = 16M

[mysql]
#no-auto-rehash # faster start of mysql but no tab completition

[isamchk]
key_buffer      = 16M

#
# * IMPORTANT: Additional settings that can override those from this file!
#   The files must end with '.cnf', otherwise they'll be ignored.
#
!includedir /etc/mysql/conf.d/
~~~~

* Restart MySQL:
  * `sudo service mysql restart`

### Create Remote User Account
***NOTE:*** This is definitely not secure and should *not* be done for a production system!

* Edit the `/etc/mysql/my.cnf` file:
  * Update the `bind-address = 127.0.0.1` to `bind-address = 0.0.0.0` and un-comment the line (if necessary)
* Restart MySQL:
  * `sudo service mysql restart`
* Log into MySQL:
  * `mysql -uroot -p`
* Execute the following commands:
  * `CREATE USER 'remoteuser'@'localhost' IDENTIFIED BY '[choose a password]';`
  * `CREATE USER 'remoteuser'@'%' IDENTIFIED BY '[the same password]';`
  * `GRANT ALL ON *.* TO 'remoteuser'@'localhost';`
  * `GRANT ALL ON *.* TO 'remoteuser'@'%';`
