
Software updates on the Ubuntu system will fail because Cockpit on Ubuntu doesn't detect a network connection.
To fix this, you must use `NetworkManager` instead of the default `networkd`.

You need to be `root` or `sudo root` for all these commands. Edit your netplan file in `/etc/netplan` and add the following line under `network:`.

```
network:
  **renderer: NetworkManager**
  ethernets:
    eno1:
      dhcp4: false
      addresses:
        - 10.13.37.2/24
      gateway4: 10.13.37.1
      nameservers:
        search: [sacredheart.lan]
        addresses: [10.13.37.3]
  version: 2
```

Remember that spaces matter in YAML files.

You will then need install `network-manager`.

```
apt install network-manager
```

Disable networkd.

```
systemctl disable systemd-networkd
```

Enable NetworkManager

```
systemctl enable network-manager
systemctl start network-manager
```

Apply your new settings

```
netplan apply
```

Profit!
