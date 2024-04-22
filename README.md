RTR-L (Debian 11)

# Настройка интерфейсов
sudo nano /etc/network/interfaces

# Добавьте следующие строки в файл:
auto ens33
iface ens33 inet static
    address 4.4.4.100
    netmask 255.255.255.0
    gateway 4.4.4.1

auto ens36
iface ens36 inet static
    address 192.168.200.254
    netmask 255.255.255.0

# Включите маршрутизацию
sudo sysctl -w net.ipv4.ip_forward=1

# Настройка PAT
sudo iptables -t nat -A POSTROUTING -o ens33 -s 192.168.200.0/24 -j MASQUERADE

# Настройка перенаправления SSH
sudo iptables -t nat -A PREROUTING -p tcp --dport 2244 -j DNAT --to-destination 192.168.200.100:22

RTR-R (Debian 11)

# Настройка интерфейсов
sudo nano /etc/network/interfaces

# Добавьте следующие строки в файл:
auto ens36
iface ens36 inet static
    address 172.16.100.254
    netmask 255.255.255.0

auto ens33
iface ens33 inet static
    address 5.5.5.100
    netmask 255.255.255.0
    gateway 5.5.5.1

# Включите маршрутизацию
sudo sysctl -w net.ipv4.ip_forward=1

# Настройка PAT
sudo iptables -t nat -A POSTROUTING -o ens33 -s 172.16.100.0/24 -j MASQUERADE

Настройка GRE-туннеля между RTR-L и RTR-R

# На RTR-L
sudo ip tunnel add Tunnel1 mode gre local 4.4.4.100 remote 5.5.5.100 ttl 255
sudo ip link set Tunnel1 up
sudo ip addr add 10.0.0.1/30 dev Tunnel1

# На RTR-R
sudo ip tunnel add Tunnel1 mode gre local 5.5.5.100 remote 4.4.4.100 ttl 255
sudo ip link set Tunnel1 up
sudo ip addr add 10.0.0.2/30 dev Tunnel1

Настройка динамической маршрутизации OSPF между RTR-L и RTR-R

# Установите quagga на обоих маршрутизаторах
sudo apt-get install quagga

# Настройка ospfd.conf на RTR-L
sudo nano /etc/quagga/ospfd.conf

# Добавьте следующие строки в файл:
router ospf
 network 192.168.200.0/24 area 0.0.0.0
 network 10.0.0.0/30 area 0.0.0.0

# Настройка ospfd.conf на RTR-R
sudo nano /etc/quagga/ospfd.conf

# Добавьте следующие строки в файл:
router ospf
 network 172.16.100.0/24 area 0.0.0.0
 network 10.0.0.0/30 area 0.0.0.0

SRV (Windows)

Настройка маршрута по умолчанию на Windows выполняется через командную строку или через графический интерфейс. Вот как это можно сделать через командную строку:

route add 0.0.0.0 mask 0.0.0.0 192.168.200.254

WEB-L (Debian 11)

# Настройка интерфейсов
sudo nano /etc/network/interfaces

# Добавьте следующие строки в файл:
auto ens33
iface ens33 inet static
    address 192.168.200.100
    netmask 255.255.255.0
    gateway 192.168.200.254

WEB-R (Debian 11)

# Настройка интерфейсов
sudo nano /etc/network/interfaces

# Добавьте следующие строки в файл:
auto ens33
iface ens33 inet static
    address 172.16.100.100
    netmask 255.255.255.0
    gateway 172.16.100.254

ISP (Debian 11)

# Настройка интерфейсов
sudo nano /etc/network/interfaces

# Добавьте следующие строки в файл:
auto ens33
iface ens33 inet static
    address 4.4.4.1
    netmask 255.255.255.0

auto ens36
iface ens36 inet static
    address 5.5.5.1
    netmask 255.255.255.0

auto ens37
iface ens37 inet static
    address 3.3.3.1
    netmask 255.255.255.0

# Включите маршрутизацию
sudo sysctl -w net.ipv4.ip_forward=1


    Установите BIND9:

sudo apt-get update
sudo apt-get install bind9 bind9utils bind9-doc

    Откройте файл настроек BIND9:

sudo nano /etc/bind/named.conf.options

    Настройте прямую зону в файле /etc/bind/named.conf.local:

sudo nano /etc/bind/named.conf.local

Добавьте следующие строки в файл:

zone "hands.lan" {
    type master;
    file "/etc/bind/db.hands.lan";
};

zone "golden.hands.lan" {
    type master;
    file "/etc/bind/db.golden.hands.lan";
};

    Создайте файлы зон:

sudo cp /etc/bind/db.local /etc/bind/db.hands.lan
sudo cp /etc/bind/db.local /etc/bind/db.golden.hands.lan

    Откройте файлы зон и добавьте записи DNS в соответствии с вашей таблицей:

sudo nano /etc/bind/db.hands.lan
sudo nano /etc/bind/db.golden.hands.lan

    После добавления всех записей перезапустите службу BIND9:

sudo systemctl restart bind9

    Проверьте статус службы BIND9:

sudo systemctl status bind9


Файл зоны для hands.lan (/etc/bind/db.hands.lan):

;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     hands.lan. root.hands.lan. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      hands.lan.
ISP     IN      A       3.3.3.1
www     IN      A       4.4.4.100
wwwt    IN      CNAME   5.5.5.100

Файл зоны для golden.hands.lan (/etc/bind/db.golden.hands.lan):

;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     golden.hands.lan. root.golden.hands.lan. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      golden.hands.lan.
internet IN     A       ISP
web-l    IN     CNAME   web-l-R
web-R    IN     CNAME   WEB-R
rtr-l    IN     A       192.168.200.254
rtr-r    IN     A       172.16.100.254
webapp-L IN     CNAME   web-l
webapp-R IN     CNAME   WEB-R
ntp      IN     CNAME   SRV
dns      IN     CNAME   SRV

Пожалуйста, убедитесь, что вы заменили ISP на соответствующий IP-адрес в вашей сети. После добавления всех записей перезапустите службу BIND9:

sudo systemctl restart bind9

Проверьте статус службы BIND9:

sudo systemctl status bind9





