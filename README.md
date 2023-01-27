# munin-node

It installs `munin-node`, configures plugins and adds it to munin server.

## Requirements

Munin server must be in inventory under "munin" name.


## Role variables

Server connection:
- `munin_node_over_ssh`: Set to `yes` if munin server should connect over ssh. File `munin_server_id_rsa.pub` is required.
- `munin_node_server_public_ip`: Public IP of munun server. Used only when `munin_node_over_ssh = yes`. Used to limit accepted ssh connections.

Server config:
- `munin_node_groups`: Groups in server config: http://guide.munin-monitoring.org/en/latest/reference/munin.conf.html#node-definitions
- `munin_node_host`: Hostname or IP used by munin server to connect. Default: `{{ ansible_host }}`.
- `munin_node_host_name`: Gives a custom name for the node, independent from `munin_node_host`. Default: `{{ ansible_host }}`.

Node plugins:
- `munin_node_disable_plugins`: List of plugins to disable. Elements are strings with plugin names.
- `munin_node_enable_plugins`: List of plugins to enable. Elements are objects:
    ```yaml
    - name: foo_bar
      # template_base in optional
      template_base: foo_
      # custom is optional
      # if yes, it'll be installed from local files
      custom: yes
      # deps is optional
      # it's a list of dependencies required by plugin
      deps:
        - something
        - python3-somelib
      # config is optional
      config:
        var1: value1
        var2: value2
      # cron is optional
      cron:
        hour: 5
        minute: 15
    ```

Variables `munin_node_(en|dis)able_plugins` can be merged from multiple group_vars by adding suffixes to variable name (see example below).


## Dependencies

None


## Example Playbook

`playbook.yml`:
```yml
- hosts: all
  roles: [munin-node]
```

`files/certbot_expiry` - a plugin downloaded from munin gallery

`inventory/inventory.yml`:
```yml
all:
  hosts:
    munin:
    host1:
    host2:
  children:
    munin_node_letsencrypt:
      hosts:
        munin:
        host1:
    munin_node_bare_metal:
      hosts:
        host1:
        host2:
```

`inventory/group_vars/all.yml`:
```yml
munin_node_enable_plugins_common:
  - name: df_abs
    config:
      env.exclude: iso9660 squashfs overlay tmpfs devtmpfs
      env.total: "off"
  - name: diskstats
    config:
      env.exclude: loop
  - name: memory
    config:
      env.apps_warning: 80%
      env.apps_critical: 95%
```

`inventory/group_vars/munin_node_letsencrypt.yml`:
```yml
munin_node_enable_plugins_letsencrypt:
  - name: certbot_expiry
    custom: yes
    config:
      user: root
```

`inventory/group_vars/munin_node_bare_metal.yml`:
```yml
munin_node_enable_plugins_bare_metal:
  - name: sensors_temp
    template_base: sensors_
    deps:
      - lm-sensors
  - name: smart_sda
    template_base: smart_
    config:
      env.ignoreexit: 96
```

## License

GPL-3.0-or-later


## Author Information

[Golem Network](https://golem.network/)
