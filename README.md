# VladimirSergeevichRU_infra
VladimirSergeevichRU Infra repository

Первое самостоятельное задание:
Есть два варианта как подключиться к someinternalhost через bastion:
1) В локальной консоли введём ssh -J appuser@<public ip bastion>  appuser@<internal ip someinternalhost>
2) В конфигурацию ~/.ssh на локальной машине добавить следущие строки:
	Host <public ip bastion>
	User appuser

	Host <internal ip someinternalhost>
	ProxyJump <public ip bastion>
	User appuser
И затем из локальной консоли выполнить подключение командой ssh <internal ip someinternalhost>


Дополнительное задание:
Для того чтобы подключиться по команде ssh someinternalhost из локальной консоли, добавим в файл /etc/hosts запись вида:
	- <internal ip someinternalhost> someinternalhost
и модифицируем ssh конфигурацию к следующему виду:

        Host <public ip bastion>
        User appuser

        Host someinternalhost
        ProxyJump <public ip bastion>
        User appuser


------------------------------------------------------------------------------------------------------------------------


Для проверки OVPN подключения к серверу bastion испольуем следующий адрес:

	public_IP = 158.160.120.207
	internal_IP = 10.128.0.27
Для проверки доступности internal host:

	someinternalhost_IP = 10.128.0.10

TLS подключение к bastion для доступа в веб панель управления VPN сервером:

	https://server.158-160-120-207.sslip.io

Учётные данные пользователя вставлены в блок конфигурации cloud-bastion.ovpn - auth-user-pass


bastion_IP = 158.160.120.207
someinternalhost_IP = 10.128.0.10
