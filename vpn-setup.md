<div id="header" align="center">
    <h1>Краткий гайд по настройке vps-сервера на примере WireGuard</h1>
</div>

* [Арендуем vps-сервер](vps-rental.md)

* [Подключаемся к vps-серверу по ssh](connecting-to-vps-by-ssh.md)

* [Генерируем ssh-ключи](copying-files-over-ssh.md)

* Обновляем пакеты на сервере

`apt update && apt upgrade -y`

* Устанавливаем **WireGuard**

`apt install -y wireguard`

* Генерируем ключи сервера

`wg genkey | tee /etc/privatekey | wg pubkey | tee /etc/wireguard/publickey`

* Проверим, как называется сетевой интерфейс

`ip a`

Скорее всего сетевой интерфейс будет **eth.0**.

* Создаем конфиг для сетевого интерфейса

`vim /etc/wireguard/wg0.conf`

Выглядеть он будет так

```
[Interface]
PrivateKey = <privatekey>
Address = 10.0.0.1/24
ListenPort = 51830
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERAGE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERAGE
```

В примере в строках PostUp и PostDown использован сетевой интерфейс **eth0** (у вас может быть и другой)
Вставляем вместо **<privatekey>** содержимое файла **/etc/wireguard/privatekey**
* Настраиваем ip форвардинг:

`echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf`

* Включаем **systemd демон** с wireguard:

```
systemctl enable wg-quick@wg0.service 
systemctl start wg-quick@wg0.service 
systemctl status wg-quick@wg0.service 
```

* Создаем ключи клиента:

`wg genkey | tee /etc/wireguard/client_privatekey | wg pubkey | tee /etc/wireguard/client_publickey`

* Добавляем в конфиг сервера клиента:

`vim /etc/wireguard/wg0.conf`

```
[Peer]
PublicKey = <client_publickey>
AllowedIPs = 10.0.0.2/32
```

Вставляем вместо **<client_publickey>** содержимое файла **/etc/wireguard/client_publickey**

Выглядеть, в итоге, он будет так

```
[Interface]
PrivateKey = <privatekey>
Address = 10.0.0.1/24
ListenPort = 51830
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERAGE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERAGE

[Peer]
PublicKey = <client_publickey>
AllowedIPs = 10.0.0.2/32
```

