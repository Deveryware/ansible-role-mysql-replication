# Ansible Role: MySQL Replication Master Slave

This role is an extension of the tleguern mysql role.
It's based on the geerlingguy's role to manage replication master slave and manage gtid features.

## Requirements

* Install mysql on your servers.You can use the tleguern roles : https://github.com/tleguern/ansible-role-mysql/
* Setup the following minimal parameters:
  * On the master
```ini

[mysqld]

log_bin =

server_id =

expire_logs_days =

```
  * On the slave:
```ini

[mysqld]

server_id =

```

## Role Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `mysql_replication_gtid` | Whether or not mysql is using gtid for the replication | `true` |
| `mysql_replication_role` | Defines whether your host is a `master` or `slave` | mandatory |
| `mysql_replication_user` | Defines replication user  | mandatory |
| `mysql_socket` | MySQL Unix domain socket used for connections | `{{ __mysql_socket }}` |
| `mysql_replication_master` | Address or fqdn of the master | mandatory (for all slave servers)|
| `mysql_master_user_gtid` | Configures the replica to use the MariaDB Global Transaction ID | `slave_pos` |

### mysql_replication_user variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `name` | Name of the replication user | mandatory |
| `host` | The 'host' part of the MySQL username | `%` |
| `password` | User's password  | mandatory |
| `priv` | MySQL privileges string in the format  | `*.*:REPLICATION SLAVE,REPLICATION CLIENT` |

## Example:

master:

```yaml

mysql_socket: /run/mysqld/mysqld.sock

mysql_replication_role: master

mysql_replication_user:

  name: replication

  host: "192.168.25.01"

  password: "{{ vault_replication_password }}"

mysql_config:

  - name: client-server

    content: |

      socket=/var/www/var/run/mysql/mysql.sock

      port=3306

  - name: mysqld

    content: |

      bind-address = 0.0.0.0

      datadir = /var/lib/mysql

      log-basename = mysqld

      tmpdir=/tmp

      general-log

      slow_query_log

      log_error = /var/log/mysql/error.log

      ## Replication conf

      log_bin = binlog-m1

      server_id = 1

      expire_logs_days = 3

      gtid-domain-id = 1

  - name: mysqld_safe

    content: |

      syslog

```

slave:

```yaml

mysql_replication_master: '192.168.31.03'

mysql_socket: /run/mysqld/mysqld.sock

mysql_replication_role: slave

mysql_config:

  - name: client-server

    content: |

      socket=/var/www/var/run/mysql/mysql.sock

      port=3306

  - name: mysqld

    content: |

      bind-address = 0.0.0.0

      datadir = /var/lib/mysql

      log-basename = mysqld

      tmpdir=/tmp

      general-log

      slow_query_log

      log_error = /var/log/mysql/error.log

      server_id = 2

  - name: mysqld_safe

    content: |

      syslog

```

## Dependencies

None.

## Example Playbook

```yaml
- hosts: dbs
  vars:
    - mysql_db_admin_password: "{{ vaulted_mysql_db_admin_password }}"
    - mysql_users:
        - name: app_user
          password: app_password
    - mysql_databases:
        - name: app_db
  roles:
    - ansible-role-mysql
    - ansible-role-mysql-replication-master-slave
```

## License

MIT / ISC

## Contributing

To get in touch with the author you can create a issue on github when requesting features.

## Author Information

This role contains code from [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
Additional work by Olivier Pouilly.
