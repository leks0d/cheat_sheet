**Установка cockpit и настройка NetworkManager**

```
apt -y install cockpit
systemctl enable cockpit.socket
systemctl start cockpit.socket
```

```
nano /etc/network/interfaces
```
`/etc/network/interfaces` не должен содержать ничего.

`nano /etc/NetworkManager/NetworkManager.conf` содержит: 

```
[main]
plugins=ifupdown,keyfile

[ifupdown]
managed=false
```
Перезагружаем сеть.
```
/etc/init.d/network-manager restart
```
