# Ansible Role: MySQL Replication Master Slave

This role is implemented as an extension to [tleguern's ansible-role-mysql][tleguern-mysql], itself a redesign of [geerlingguy's][geerlingguy-mysql] only containing the bare minimum.

The content evolved from Jeff Geerling original replication tasks and now include GTID support.

## Requirements

An ansible role dedicated to the installation of MySQL/MariaDB such as [tleguern.mysql][tleguern-mysql].

The following set of parameters must be configured on the master:

```ini
[mysqld]
log_bin =
server_id =
expire_logs_days =
```

And also on the slave:

```ini
[mysqld]
server_id =
```

## Role Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `mysql_replication_gtid` | Whether or not mysql is using gtid for the replication | `true` |
| `mysql_replication_role` | Defines whether a host is a `master` or a `slave` | mandatory |
| `mysql_replication_user` | Defines the replication user | mandatory |
| `mysql_socket` | MySQL Unix domain socket used for connections | `{{ __mysql_socket }}` |
| `mysql_replication_master` | IP address or FQDN of the master | mandatory (for all slave servers)|
| `mysql_master_user_gtid` | The value passed to [`master_use_gtid`][master-use-gtid] | `slave_pos` |

### `mysql_replication_user`

This variable represents the replication user and its associated rights as a YAML dictionary using the following structure:

| Variable | Description | Default |
|----------|-------------|---------|
| `name` | Name of the replication user | mandatory |
| `host` | The 'host' part of the MySQL username | `%` |
| `password` | User's password  | mandatory |
| `priv` | MySQL privileges string in the format  | `*.*:REPLICATION SLAVE,REPLICATION CLIENT` |

Example:

```yaml
mysql_replication_user:
  name: replication
  host: 192.168.25.1
  password: "{{ vault_replication_password }}"
```

## Dependencies

None.

## Example Playbook

```yaml
- hosts: databases
  vars:
    - mysql_socket: /run/mysqld/mysqld.sock
    - mysql_db_admin_password: "{{ vaulted_mysql_db_admin_password }}"
    - mysql_users:
        - name: app_user
          password: app_password
    - mysql_databases:
        - name: app_db
  roles:
    - role: ansible-role-mysql

- hosts: database_primary
  vars:
    - mysql_replication_role: master
    - mysql_replication_user:
        name: replication
        host: 192.168.25.1
        password: "{{ vault_replication_password }}"
    - mysql_config:
        - name: mysqld
          content: |
            server_id = 1
            expire_logs_days = 3
            gtid-domain-id = 1
            log_bin = binlog-m1
  roles:
    - role: ansible-role-mysql
    - role: ansible-role-mysql-replication-master-slave

- hosts: database_secondary
  vars:
    - mysql_replication_role: slave
    - mysql_replication_master: 192.168.31.3
    - mysql_config:
        - name: mysqld
          content: |
            server_id = 2
  roles:
    - role: ansible-role-mysql
    - role: ansible-role-mysql-replication-master-slave
```

## License

MIT / ISC

## Contributing

To get in touch with the author you can create a issue on github when requesting features.

## Author Information

This role contains code from [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
Additional work by Olivier Pouilly.

[tleguern-mysql]: https://git.sr.ht/~tleguern/ansible-role-mysql
[geerlingguy-mysql]: https://github.com/geerlingguy/ansible-role-mysql
[master-use-gtid]: https://docs.ansible.com/ansible/latest/collections/community/mysql/mysql_replication_module.html#parameter-master_use_gtid
