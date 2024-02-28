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
```
mcedit /etc/net/ifaces/(интерфейс)/options
```
```
mcedit /etc/net/ifaces/(интерфейс)/ipv4address
```
```
mcedit /etc/net/ifaces/(интерфейс)/ipv4route
```
```
mkdir /etc/net/ifaces/(интерфейс)
```
```
cp /etc/net/ifaces/(интерфейс)/options /etc/net/ifaces/(интерфейс)/options
```
```
service network restart
```
```
hostnamectl set-hostname (имя);exec bash
```
