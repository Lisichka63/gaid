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
    up ip route add default via 192.168.1.1 dev ens33

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
    up ip route add default via 192.168.2.1 dev ens33

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

# Установите пакет iptables
apt install iptables

# Включить IP-преадресацию
echo 1 > /proc/sys/net/ipv4/ip_forward

# Настроить NAT
iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
iptables -A FORWARD -i ens33 -o ens36 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i ens36 -o ens33 -j ACCEPT
iptables -A FORWARD -i ens33 -o ens37 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i ens37 -o ens33 -j ACCEPT

На RTR:

#Установите пакет iptables
apt install iptables

# Включить IP-преадресацию
echo 1 > /proc/sys/net/ipv4/ip_forward

# Настроить NAT(PAT)
iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
iptables -A FORWARD -i ens33 -o ens36 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i ens36 -o ens33 -j ACCEPT

На LTL:

# Установите пакет iptables
apt install iptables

# Включить IP-преадресацию
echo 1 > /proc/sys/net/ipv4/ip_forward

# Настроить NAT(PAT)
iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
iptables -A FORWARD -i ens33 -o ens36 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i ens36 -o ens33 -j ACCEPT

На RTR и LTL и ISP:

# Для сохранения настроек iptables при перезапуске системы, вы можете использовать утилиту iptables-persistent. Вот как вы можете это сделать:

Установите пакет iptables-persistent:
sudo apt-get install iptables-persistent

Во время установки, система спросит вас, хотите ли вы сохранить текущие правила iptables. Выберите “Yes” для сохранения текущих правил.

Если вы внесли изменения в правила iptables после установки iptables-persistent, вы можете сохранить текущие правила вручную:
sudo netfilter-persistent save

При перезагрузке системы, iptables-persistent автоматически загрузит сохраненные правила.

На WEB-R

#echo 1 > /proc/sys/net/ipv4/ip_forward

На WEB-L

#echo 1 > /proc/sys/net/ipv4/ip_forward


# Создание GRE-туннеля:

На RTR-L:

sudo ip tunnel add Tunnel1 mode gre local 192.168.1.2 remote 192.168.2.2 ttl 255
sudo ip link set Tunnel1 up
sudo ip addr add 10.0.0.1/30 dev Tunnel1

На RTR-R:

sudo ip tunnel add Tunnel1 mode gre local 192.168.2.2 remote 192.168.1.2 ttl 255
sudo ip link set Tunnel1 up
sudo ip addr add 10.0.0.2/30 dev Tunnel1

# Установка StrongSwan (для IPsec): На каждом из ваших роутеров выполните следующую команду для установки StrongSwan:

sudo apt-get install strongswan

# Настройка IPsec: Создайте или отредактируйте файл /etc/ipsec.conf на каждом роутере. Вот пример того, что вы можете включить в этот файл:

На RTR-L:

conn mytunnel
    auto=start
    left=192.168.1.2
    right=192.168.2.2
    authby=secret
    type=transport
    keyexchange=ikev2

На RTR-R:

conn mytunnel
    auto=start
    left=192.168.1.2
    right=192.168.2.2
    authby=secret
    type=transport
    keyexchange=ikev2

# Настройка предварительно согласованного ключа (PSK): Ключи для IPsec определяются в файле /etc/ipsec.secrets. В этом файле вы указываете предварительно согласованный ключ (PSK) для вашего туннеля. Вот пример конфигурации:

На RTR-L:

192.168.1.2 192.168.2.2 : PSK "your_pre_shared_key"

На RTR-R:

192.168.1.2 192.168.2.2 : PSK "your_pre_shared_key"

В этом примере мы определяем PSK для идентификаторов 4.4.4.100 и 5.5.5.100. Замените "your_pre_shared_key" на ваш предварительно согласованный ключ. Этот ключ должен быть одинаковым на обоих роутерах.

Перезапуск IPsec: После того, как вы настроили IPsec и ключи, вы должны перезапустить IPsec с помощью команды sudo systemctl restart ipsec.








