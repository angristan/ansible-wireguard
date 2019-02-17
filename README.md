# Ansible role for Wireguard

This role will install WireGuard from the Sid repo on Debian 9 and from the PPA on Ubuntu.

The role's scope is an inventory group, from which it will define peers (see the example below).

The role does **not** handle keys. You can create a pair with:

```sh
private_key=$(wg genkey)
public_key=$(echo $private_key | wg pubkey)
echo "private key: $private_key"
echo "public key: $public_key"
```

Then put these in `wireguard_public_key` and `wireguard_private_key` for each host (the use of *ansible-vault* is recommended).

Bu default, the endpoint for each peer is its default IPv4 address from the Ansible facts. This can be overwritten with `wireguard_endpoint`.

## Example playbook

In this example we'll use `host1` and `host2`, which are part of the `web` inventory group.

The inventory:

```ini
[web]
host1
host2
```

The vars in `group_vars/web.yml`:

```yaml
wireguard_listen_port: 1194
wireguard_interface: wg0
wireguard_group: web
```

`host_vars/host1.yml`:

```yaml
wireguard_address: 10.0.0.1
wireguard_public_key: 'Z3YGMbf/RBj0dg3bOhV2ijxx05OQ0s3MlI5u60kWlng='
wireguard_private_key: 'GGp+zuAJwzHm3Q+EimK0tYA3jF7ipnn3GwdIzbBj3V8='
```

`host_vars/host2.yml`:

```yaml
wireguard_address: 10.0.0.2
wireguard_public_key: 'oIQu/u/amX4WsvX/MeSSGtL8hxbUdtX3P4JGICV3bRw='
wireguard_private_key: 'uCpU57hdstHu3JarIm8XbLxbPbNq4gtGSdnTNi3ksl8='
```

The playbook

```yaml
---

- hosts: web
  roles:
    - name: wireguard
      tags: wireguard
```

The role will:

- Install wireguard
- Enable IPv4 forwarding
- Setup the configuration file in `/etc/wireguard/{{ wireguard_interface }}.conf` with `[Interface]` and `[Peer]` as needed
- Update `/etc/hosts` with the inventory_names + their WireGuard IP
- Enable and start the WireGuard interface

In our example, the role will append this to `/etc/hosts` automatically:

On host1:

```
10.0.0.2 host2
```

On host2:

```
10.0.0.1 host1
```
