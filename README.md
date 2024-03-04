# ALT-DEMO-2024
|Имя устройства  |Интерфейс           |IPv4/IPv6       |Маска/Префикс   |Шлюз                  |                       
|  ------------- | -------------      | -------------  |  ------------- |  -------------       |                    
|ISP             |ens160              |10.12.11.12     |/24             |10.12.11.254          |      
|                |ens192              |192.168.0.162   |/30             |                      |
|                |ens224              |192.168.0.166   |/30             |                      |
|HQ-R            |ens160              |192.168.0.161   |/30             |192.168.0.162         |                                   
|                |ens192              |192.168.0.1     |/25             |                      |
|BR-R            |ens160              |192.168.0.165   |/30             |192.168.0.166         |                                  
|                |ens192              |192.168.0.129   |/27             |                      |
|HQ-SRV          |ens192              |192.168.0.126   |/25             |192.168.0.1           |                                   
|BR-SRV          |ens192              |192.168.0.158   |/27             |192.168.0.129         |   

## 1. Настройка интерфейсов
### Поменять "dhcp" на "static"
```
mcedit /etc/net/ifaces/(интерфейс)/options
```
### Настройка ipv4address
```
mcedit /etc/net/ifaces/(интерфейс)/ipv4address
```
### Настройка шлюза
```
mcedit /etc/net/ifaces/(интерфейс)/ipv4route
```
### Создание каталога
```
mkdir /etc/net/ifaces/(интерфейс)
```
### Копия конфы из одного интерфейса в другой
```
cp /etc/net/ifaces/(интерфейс)/options /etc/net/ifaces/(интерфейс)/options
```
### Перезагрузка интерфейсы
```
service network restart
```
### Поменять имя
```
hostnamectl set-hostname (имя);exec bash
```
### Включил маршрутизацию
```
mcedit /etc/net/sysctl.conf
```
```
net.ipv4.ip_forward=1
```
### Установка nmtui
```
apt-get update && apt-get install -y NetworkManager-{daemon,tui}
```
### Запускаем и добавляем в автозагрузку службу "NetworkManager"
```
systemctl enable --now NetworkManager
```
### Для того, чтобы интерфейсы стали видимы в nmtui или nmcli - необходимо поправить параметр в файле /etc/net/ifaces/<ИМЯ_ИНТЕРФЕЙСА/options
```
sed -i "s/NM_CONTROLLED=no/NM_CONTROLLED=yes/g" /etc/net/ifaces/ens33/options
```
### Перезапускаем службы "network" и "NetworkManager"
```
systemctl restart network
```
```
systemctl restart NetworkManager
```
### Заходим в интерфейс
```
nmtui
```
### Далее (Добавить - IP tunnel)
### HQ-R
![Топология](https://github.com/Barmenkov/demo2024/blob/main/%D0%91%D0%B5%D0%B7%D1%8B%D0%BC%D1%8F%D0%BD%D0%BD%D1%8B%D0%B9.png)
### BR-R
![Топология](https://github.com/Barmenkov/demo2024/blob/main/%D0%91%D0%B5%D0%B7%D1%8B%D0%BC%D1%8F%D0%BD%D0%BD%D1%8B%D0%B92.png)
```
###
###
###
###
###
