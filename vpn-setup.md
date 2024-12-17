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
Private
```
