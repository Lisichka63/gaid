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


