# Ansible Role: MySQL Replication

This role is implemented as an extension to [tleguern's ansible-role-mysql][tleguern-mysql], itself a redesign of [geerlingguy's][geerlingguy-mysql] only containing the bare minimum.

The content evolved from Jeff Geerling original replication tasks and now include GTID support.

## Requirements

An ansible role dedicated to the installation of MySQL/MariaDB such as [tleguern.mysql][tleguern-mysql].

The following set of parameters must be configured on the primary(s):

```ini
[mysqld]
log_bin =
server_id =
expire_logs_days =
```

## Role Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `mysql_replication_gtid` | Whether or not mysql is using gtid for the replication | `true` |
| `mysql_replication_user` | Defines the replication user | mandatory |
| `mysql_socket` | MySQL Unix domain socket used for connections | `{{ __mysql_socket }}` |
| `mysql_replication_primary` | IP address or FQDN of the primary | mandatory |
| `mysql_primary_use_gtid` | The value passed to [`primary_use_gtid`] | `slave_pos` |
| `mysql_replication_role_primary` | Define if the server is the primary | `false` |
| `mysql_replication_role_replica` | Define if the server is a replica | `false` |

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
  host: 192.168.25.%
  password: "{{ vault_replication_password }}"
```

## Dependencies

None.

## Example of a multi primary configuration

Variable DB1:

```yaml
mysql_config:
...
  - name: mysqld
    content: |
      server_id = 1
      expire_logs_days = 3
      gtid-domain-id = 1
      log_bin = binlog-m1
...
mysql_replication_user:
  name: replication
  host: 192.68.25.%
  password: "P@ssW0rD!"
mysql_replication_primary: db2.exemple.com
mysql_replication_role_primary: true
mysql_replication_role_replica: true
```

Variable DB2:

```yaml
mysql_config:
...
  - name: mysqld
    content: |
      server_id = 2
      expire_logs_days = 3
      gtid-domain-id = 1
      log_bin = binlog-m1
...
mysql_replication_user:
  name: replication
  host: 192.68.25.%
  password: "P@ssW0rD!"
mysql_replication_primary: db1.exemple.com
mysql_replication_role_primary: true
mysql_replication_role_replica: true
```

Variable DB3:

```yaml
mysql_config:
...
  - name: mysqld
    content: |
      server_id = 3
...
mysql_replication_user:
  name: replication
  host: 192.68.25.%
  password: "P@ssW0rD!"
mysql_replication_primary: db1.exemple.com
mysql_replication_role_replica: true
```

Playbook:

```yaml
- hosts: DBS
  roles:
    - role: ansible-role-mysql
    - role: ansible-role-mysql-replication

```

## License

MIT / ISC

## Contributing

To get in touch with the author you can create a issue on github when requesting features.

## Author Information

This role was created in 2021 by Olivier Pouilly on the behalf of Deveryware.

[tleguern-mysql]: https://git.sr.ht/~tleguern/ansible-role-mysql
[geerlingguy-mysql]: https://github.com/geerlingguy/ansible-role-mysql
