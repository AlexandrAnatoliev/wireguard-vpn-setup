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

* Генерируем ключи **wireguard**-сервера

`wg genkey | tee /etc/wireguard/privatekey | wg pubkey | tee /etc/wireguard/publickey`

Сгенерированные публичный и приватный ключи будут сохранены в соответствующих файлах

```
/etc/
    wireguard/
        privatekey
        publickey
```

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
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

Файл с конфигурацией **wireguard**-сервера находится в этой же папке

```
/etc/
    wireguard/
        privatekey
        publickey
        wg0.conf
```

В примере в строках PostUp и PostDown использован сетевой интерфейс **eth0** (у вас может быть и другой)
Вставляем вместо **<privatekey>** содержимое файла **/etc/wireguard/privatekey**

* Настраиваем ip форвардинг:

`echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf`

* Включаем **systemd демон** с **wireguard**:

```
systemctl enable wg-quick@wg0.service 
systemctl start wg-quick@wg0.service 
systemctl status wg-quick@wg0.service 
```

* Создаем ключи клиента:

`wg genkey | tee /etc/wireguard/client_privatekey | wg pubkey | tee /etc/wireguard/client_publickey`

```
/etc/
    wireguard/
        client_privatekey
        client_publickey
        privatekey
        publickey
        wg0.conf
```

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
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <client_publickey>
AllowedIPs = 10.0.0.2/32
```

* Перезагружаем **systemctl** сервис с **wireguard**

```
systemctl restart wg-quick@wg0
systemctl status wg-quick@wg0
```

* На ЛОКАЛЬНОЙ машине (клиенте) создаем текстовый файл с конфигом клиента

`vim client_wb.conf`

```
[Interface]
PrivateKey = <client_privatekey>
Address = 10.0.0.2/32
DNS = 8.8.8.8

[Peer]
PublicKey = <publickey>
Endpoint = <server-ip>:51830
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 20
```

Вставляем вместо **<client_privatekey>** содержимое файла **/etc/wireguard/client_privatekey**, 

вместо **<publickey>** - **/etc/wireguard/publickey**,

**<server-ip>** заменяем на **ip** сервера.

* Этот конфигурационный файл открываем в **wireguard-клиенте** телефона или компьютера.

