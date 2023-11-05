# VladimirSergeevichRU_infra
VladimirSergeevichRU Infra repository

## Homework 3. Знакомство с облачной инфраструктурой Yandex.Cloud
```
bastion_IP = 158.160.120.207
someinternalhost_IP = 10.128.0.10
```
### Создание VM и подключение к ним по ssh
На локальной машине сгенерировал ssh ключ для дальнейшего использования в cloud.yandex
```
ssh-keygen -t rsa -f ~/.ssh/appuser -C appuser -P ""
```
Далее создал две VM в итоге получаем:
![image](https://github.com/Otus-DevOps-2023-09/VladimirSergeevichRU_infra/assets/123881243/0e87512e-356e-4735-acb2-070a3138600d)

Подключился к bastion по ssh:
```
ssh -i ~/.ssh/appuser appuser@158-160-120-207
```
Далее подключимся к someinternalhost через bastion из локальной консоли. Варианта два:
```
1) В локальной консоли введём ssh -J appuser@<public ip bastion>  appuser@<internal ip someinternalhost>
2) В конфигурацию ~/.ssh на локальной машине добавить следущие строки:
	Host <public ip bastion>
	User appuser

	Host <internal ip someinternalhost>
	ProxyJump <public ip bastion>
	User appuser
И затем из локальной консоли выполнить подключение командой ssh <internal ip someinternalhost>
```

## Дополнительное задание.

Подключиться к someinternalhost используя агента в одну команду можно следующим образом:
```
ssh -i ~/.ssh/appuser -A appuser@158.160.120.207 ssh 10.128.0.10
```

Для того чтобы подключиться по команде ssh someinternalhost (по алиасу) из локальной консоли, добавим в файл /etc/hosts запись вида:
	```<internal ip someinternalhost> someinternalhost```
и модифицируем ssh конфигурацию к следующему виду:
```
        Host <public ip bastion>
        User appuser

        Host someinternalhost
        ProxyJump <public ip bastion>
        User appuser
```
Также есть второй вариант реализовать задействовав только настройку ssh config:
```
Host bastion
    HostName <public ip bastion>
    User appuser
    ForwardAgent yes
    IdentityFile ~/.ssh/appuser

Host someinternalhost
    HostName <internal ip someinternalhost>
    ProxyJump bastion
    User appuser
    ForwardAgent yes
    IdentityFile ~/.ssh/appuser
```
------------------------------------------------------------------------------------------------------------------------
### VPN-сервер для серверов в Yandex.Cloud

Установка pritunl и запуск

```
sudo tee /etc/apt/sources.list.d/pritunl.list << EOF
deb http://repo.pritunl.com/stable/apt jammy main
EOF

# Import signing key from keyserver
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv 7568D9BB55FF9E5287D586017AE645C0CF8E292A
# Alternative import from download if keyserver offline
curl https://raw.githubusercontent.com/pritunl/pgp/master/pritunl_repo_pub.asc | sudo apt-key add -

sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list << EOF
deb https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse
EOF

wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -

sudo apt update
sudo apt --assume-yes upgrade

# WireGuard server support
sudo apt -y install wireguard wireguard-tools

sudo ufw disable

sudo apt -y install pritunl mongodb-org
sudo systemctl enable mongod pritunl
sudo systemctl start mongod pritunl
```
В результате получаем установленную MongoDB и pritunl.

Зашел на https://158.160.120.207/setup, выполнил инструкции установки.
Создал организацию otus, пользователя test, и сервер OpenVPN,после привязал его к организации и запустил.
[Подробнее](https://docs.pritunl.com/docs/connecting).

После запуска, скачал конфигурационный файл юзера на странице Users -> Click to download profile.
Для проверки OVPN подключения к серверу bastion использовал клиента Ubnuntu из nm. После установления туннеля зашёл на someinternalhost по ssh:
`ssh -i ~/.ssh/appuser appuser@10.128.0.10`


С помощью сервиса [https://sslip.io/](https://sslip.io/)  dns, который при запросе имени хоста со встроенным IP-адресом возвращает этот самый IP-адрес. В настройках pritunl добавил домен Lets Encrypt Domain, после чего стало возможным подключаться к веб управлению VPN сервером через TLS:

```
https://server.158-160-120-207.sslip.io
```
