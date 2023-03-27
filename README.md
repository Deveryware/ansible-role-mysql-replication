# Ansible Role: MySQL Replication

This role configures mysql or mariadb replication with GTID if possible.

## Requirements

At least two working mysql servers with binary logs enabled.

## Role Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `mysql_replication_user` | Defines the replication user | mandatory |
| `mysql_socket` | MySQL Unix domain socket used for connections | `/run/mysqld/mysqld.sock` |
| `mysql_replication_primary` | IP address or FQDN of the primary | mandatory |
| `mariadb_primary_use_gtid` | The value passed to [`primary_use_gtid`] (MariaDB only) | `slave_pos` |

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

collection community.mysql >= 3.1.0 (included in ansible >= 6.x)

## Example of a bi-directional replication

Inventory:

```yaml
all:
  children:
    db:
      vars:
        mysql_replication_user:
          name: replication
          host: 192.68.25.%
          password: "P@ssW0rD!"
      hosts:
        db1.exemple.com:
          mysql_replication_primary: db2.exemple.com
        db2.exemple.com:
          mysql_replication_primary: db1.exemple.com
```

Playbook:

```yaml
- hosts: db
  roles:
    - role: ansible-role-mysql-replication
```

## License

MIT / ISC

## Contributing

To get in touch with the author you can create a issue on github when requesting features.

## Author Information

This role was created in 2021 by Olivier Pouilly on the behalf of Deveryware.
