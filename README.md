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
## 2. Тунель
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
### Для BR-R
```
nmcli connection modify BR-R ip-tunnel.ttl 64
```
```
ip r add 192.168.0.0/25 dev gre1
```
### Для HQ-R
```
nmcli connection modify HQ-R ip-tunnel.ttl 64
```
```
ip r add 192.168.0.128/27 dev gre1
```
## 3. FRR
### Установка frr
```
apt-get update && apt-get install -y frr
```
```
vim /etc/frr/daemons
```
### Поменять значение
```
переводим ospfd=no в ospfd=yes - для OSPFv2 (IPv4)
```
```
переводим ospf6d=no в ospfd6=yes - для OSPFv3 (IPv6)
```
### Включаем и добавляем в автозагрузку службу frr:
```
systemctl enable --now frr
```
### Настраиваем OSPFv2 - переходим в интерфейс frr при помощи "vtysh" на HQ-R:
![](https://github.com/Barmenkov/ALT-DEMO-2024/blob/main/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA.PNG)
### на BR-R
![](https://github.com/Barmenkov/ALT-DEMO-2024/blob/main/2.PNG)
###
## 3. NAT на ISP, HQ-R, BR-R
### Установка
```
apt-get -y install firewalld
```
### Автозагрузка
```
systemctl enable --now firewalld
```
### Правило к исходящим пакетам (тот интерфейс который смотрит во внеш. сеть например на исп 192)
```
firewall-cmd --permanent --zone=public --add-interface=ens__
```
### Правило к входящим пакетам (тот интерфейс который смотрит во внутрен. сеть например на исп 224 и 256)
```
firewall-cmd --permanent --zone=trusted --add-interface=ens__
```
### Включение NAT
```
firewall-cmd --permanent --zone=public --add-masquerade
```
### Cохранение правил
```
firewall-cmd --reload
```
## 4. Настройка локальных учетных записей.
### Добавление пользователя
```
useradd admin -m -c "Admin" -U 
```
### Поменять пароль
```
passwd admin
```
### Просмотр пользователей
```
vim /etc/passwd
```
## 5. DHCP
### Установка DHCP
```
apt-get install -y dhcp-server
```
### Вошёл в файл
```
mcedit /etc/sysconfig/dhcpd 
```
### Указал интерфейс который смотрит на SRV
```
DHCPDARGS=ens224
```
### Далее следует настройка раздачи адресов, а для этого захожу в файл
```
mcedit /etc/dhcp/dhcpd.conf
```
### Прописываю конфиг
```
# dhcp.conf

default-lease-time 6000;
max-lease-time 72000;

subnet 192.168.0.0 netmask 255.255.255.128 {
range 192.168.0.10 192.168.0.125;
option routers 192.168.0.1;
}
```
### Повторяю конфиг в файле /etc/dhcp/dhcpd.conf.example Запускаю и добавляю в автозагрузку слжубу
```
systemctl enable --now dhcpd
```
## 6. Измерение пропускной способности сети между двумя узлами
### Установка пакета
```
apt-get install -y iperf3
```
### Запуск службы
```
systemctl enable --now iperf3
```
### Запуск iperf3 в качестве клиента
```
iperf3 -c 192.168.0.162
```
## 7. Настройка подключения по SSH для удаленного конфигурирования устройства
### На HQ-SRV меняем порт с 22 на 2222
```
sed -i "s/#Port 22/Port 2222/g" /etc/openssh/sshd_config
```
### Перезагружаем службу sshd
```
systemctl restart sshd
```
### Проверка
```
ss -tlpn | grep sshd
```
### На HQ-R устанавливаем nftables
```
apt-get install -y nftables
```
### Включаем и добавляем в автозагрузку службу nftables
```
systemctl enable --now nftables
```
### Создаём правило
```
nft add table inet nat
```
### Добавляем цепочку в таблицу
```
nft add chain inet nat prerouting '{ type nat hook prerouting priority 0; }'
```
### Добавляем ещё одно правило
```
nft add rule inet nat prerouting ip daddr 192.168.0.161 tcp dport 22 dnat to 192.168.0.10:2222
```
### Сохраняем правила
```
nft list ruleset | tail -n 7 | tee -a /etc/nftables/nftables.nft
```
### Перезапускаем службу
```
systemctl restart nftables
```
### Проверка на BR-R
```
ssh admin@(адрес смотрящий в ISP)
password
hostname
```
## 8. Настройка контроль доступа до HQ-SRV по SSH
### Установка nftables
```
apt-get update && apt-get install -y nftables
```
### Включаю и добавляю в автозагрузку
```
systemctl enable --now nftables
```
### Добавляю правило
```
nft add rule inet filter input ip saddr 192.168.0.170 tcp dport 2222 counter drop
nft add rule inet filter input ip saddr 192.168.0.0/30 tcp dport 2222 counter drop
```
### Проверка
```
nft list ruleset
```
### В файле /etc/nftables/nftables.nft удаляю незакомментированные строчки Отправляю результат
```
nft list ruleset | tee -a /etc/nftables/nftables.nft
```
### Перезапускаю
```
systemctl restart nftables
```
### Проверка заключается в том, что подключиться не получится и будет выдано сообщение: "connect to host 192.168.0.170 port 2222: Connection timed out
```
