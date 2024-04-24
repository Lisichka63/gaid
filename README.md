Вы можете настроить IP-адреса и шлюзы в файле /etc/network/interfaces. Вот пример того, как вы можете настроить это для каждого устройства:

На ISP:

# /etc/network/interfaces
auto ens33
iface ens33 inet dhcp

auto ens36
iface ens36 inet static
    address 192.168.1.1
    netmask 255.255.255.0

auto ens37
iface ens37 inet static
    address 192.168.2.1
    netmask 255.255.255.0

На RTR:

# /etc/network/interfaces
auto ens33
iface ens33 inet static
    address 192.168.1.2
    netmask 255.255.255.0
    gateway 192.168.1.1

auto ens36
iface ens36 inet static
    address 192.168.3.1
    netmask 255.255.255.0

На LTL:

# /etc/network/interfaces
auto ens33
iface ens33 inet static
    address 192.168.2.2
    netmask 255.255.255.0
    gateway 192.168.2.1

auto ens36
iface ens36 inet static
    address 192.168.4.1
    netmask 255.255.255.0

На WEB-L:

# /etc/network/interfaces
auto ens33
iface ens33 inet static
    address 192.168.4.2
    netmask 255.255.255.0
    gateway 192.168.4.1

На WEB-R:

# /etc/network/interfaces
auto ens33
iface ens33 inet static
    address 192.168.3.2
    netmask 255.255.255.0
    gateway 192.168.3.1

После внесения изменений, перезапустите сетевой интерфейс с помощью команды sudo systemctl restart networking.

Для настройки iptables и NAT в вашем сценарии, вы можете следовать следующим шагам. Пожалуйста, замените ens33, ens36 и ens37 на соответствующие интерфейсы в вашей системе.

На ISP:

# Включить IP-преадресацию
echo 1 > /proc/sys/net/ipv4/ip_forward

# Настроить NAT
iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
iptables -A FORWARD -i ens33 -o ens36 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i ens36 -o ens33 -j ACCEPT
iptables -A FORWARD -i ens33 -o ens37 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i ens37 -o ens33 -j ACCEPT

На RTR:

# Включить IP-преадресацию
echo 1 > /proc/sys/net/ipv4/ip_forward

# Настроить NAT
iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
iptables -A FORWARD -i ens33 -o ens36 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i ens36 -o ens33 -j ACCEPT

На LTL:

# Включить IP-преадресацию
echo 1 > /proc/sys/net/ipv4/ip_forward

# Настроить NAT
iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
iptables -A FORWARD -i ens33 -o ens36 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i ens36 -o ens33 -j ACCEPT

На WEB-L и WEB-R:

# Установить маршрут по умолчанию
route add default gw <IP_RTR_or_LTL>






