# gaid

Если у вас на роутере с NAT три интерфейса, один из которых подключен к глобальной сети, а два других - к роутерам, вы можете настроить маршрутизацию следующим образом:

    Настройка сетевых интерфейсов на роутере. Войдите в роутер через SSH или консоль и отредактируйте файл /etc/network/interfaces:

sudo nano /etc/network/interfaces

Добавьте следующие строки, заменив eth0, eth1, eth2, 192.0.2.1, 10.0.0.1 и 10.0.1.1 на соответствующие значения:

auto eth0
iface eth0 inet static
    address 192.0.2.1
    netmask 255.255.255.0
    gateway 192.0.2.254

auto eth1
iface eth1 inet static
    address 10.0.0.1
    netmask 255.255.255.0

auto eth2
iface eth2 inet static
    address 10.0.1.1
    netmask 255.255.255.0


Для сохранения правил iptables после перезагрузки в Linux, вы можете использовать утилиту iptables-persistent. Вот как это сделать:

    Установите iptables-persistent: Вы можете установить iptables-persistent, используя следующую команду:

sudo apt-get install iptables-persistent

Во время установки вам будет предложено сохранить текущие правила iptables. Выберите “Yes” для сохранения текущих правил.

    Сохраните текущие правила iptables: Если вы уже установили iptables-persistent и хотите сохранить текущие правила, вы можете использовать следующие команды:

sudo sh -c 'iptables-save > /etc/iptables/rules.v4'
sudo sh -c 'ip6tables-save > /etc/iptables/rules.v6'

Эти команды сохранят текущие правила iptables и ip6tables в файлы rules.v4 и rules.v6 соответственно.

Загрузите сохраненные правила при загрузке системы: iptables-persistent автоматически загрузит сохраненные правила при загрузке системы.


    Включение маршрутизации на роутере. Отредактируйте файл /etc/sysctl.conf:

sudo nano /etc/sysctl.conf

Раскомментируйте или добавьте следующую строку:

net.ipv4.ip_forward=1

Примените изменения:

sudo sysctl -p

****************************************************************************************************************************************************
Настройка NAT на роутере. Добавьте следующие правила iptables, заменив eth0, eth1 и eth2 на соответствующие значения:

iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
iptables -A FORWARD -i ens33 -o ens36 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i ens36 -o ens33 -j ACCEPT
iptables -A FORWARD -i ens33 -o ens37 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i ens37 -o ens33 -j ACCEPT

Это позволит вашему роутеру перенаправлять трафик между глобальной сетью и двумя другими роутерами. Пожалуйста, учтите, что эти команды являются общими и могут потребовать дополнительной настройки в зависимости от вашей конкретной сетевой конфигурации. Удачи в настройке вашей сети! Если у вас возникнут проблемы, не стесняйтесь обращаться за помощью.

sudo systemctl restart networking

Проверьте настройки DNS. Если у вас есть проблемы с разрешением имен, это может мешать вашему роутеру подключаться к Интернету. Вы можете проверить текущие настройки DNS в файле /etc/resolv.conf.


Спасибо за предоставленную информацию. Исходя из вашей конфигурации, вам, возможно, придется добавить следующие правила iptables на RTR-R:

# Принять входящий трафик от Windows-сервера
sudo iptables -A INPUT -i ens36 -j ACCEPT

# Принять исходящий трафик к ISP
sudo iptables -A OUTPUT -o ens33 -j ACCEPT

# Принять прямой трафик от Windows-сервера к ISP
sudo iptables -A FORWARD -i ens36 -o ens33 -j ACCEPT

# Принять обратный трафик от ISP к Windows-серверу
sudo iptables -A FORWARD -i ens33 -o ens36 -m state --state RELATED,ESTABLISHED -j ACCEPT

# Настроить PAT
iptables -t nat -A POSTROUTING -o ens33 -s 192.168.200.0/24 -j MASQUERADE

Эти команды позволят RTR-R принимать входящий трафик от Windows-сервера и отправлять исходящий трафик к ISP. 
Они также позволят RTR-R принимать прямой трафик от ISP к Windows-серверу и обратный трафик от Windows-сервера к ISP.
******************************************************************************************************************************************




Для сохранения правил iptables после перезагрузки в Linux, вы можете использовать утилиту iptables-persistent. Вот как это сделать:

    Установите iptables-persistent: Вы можете установить iptables-persistent, используя следующую команду:

sudo apt-get install iptables-persistent

Во время установки вам будет предложено сохранить текущие правила iptables. Выберите “Yes” для сохранения текущих правил.

    Сохраните текущие правила iptables: Если вы уже установили iptables-persistent и хотите сохранить текущие правила, вы можете использовать следующие команды:

sudo sh -c 'iptables-save > /etc/iptables/rules.v4'
sudo sh -c 'ip6tables-save > /etc/iptables/rules.v6'

Эти команды сохранят текущие правила iptables и ip6tables в файлы rules.v4 и rules.v6 соответственно.

Загрузите сохраненные правила при загрузке системы: iptables-persistent автоматически загрузит сохраненные правила при загрузке системы.








Ваши инструкции связаны с настройкой статического маршрута по умолчанию и динамической трансляции портов (PAT) на маршрутизаторах RTR-L и RTR-R. Вот базовые команды для выполнения этих задач на Debian:

1.5. Настройка статического маршрута по умолчанию:

ISP

apt isntall iptables

iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE

# На RTR-L и RTR-R
sudo ip route add default via <IP-адрес-шлюза>            

sudo ip route add default via 5.5.5.1

Замените <IP-адрес-шлюза> на IP-адрес вашего шлюза по умолчанию.

1.6. Настройка динамической трансляции портов (PAT):
%%
На RTR-L:

# Установите iptables
sudo apt-get install iptables

# Настройте PAT
sudo iptables -t nat -A POSTROUTING -o <Имя-исходящего-интерфейса> -s 192.168.200.0/24 -j MASQUERADE
%%
%%
На RTR-R:

# Установите iptables
sudo apt-get install iptables

# Настройте PAT
sudo iptables -t nat -A POSTROUTING -o <Имя-исходящего-интерфейса> -s 172.16.100.0/24 -j MASQUERADE

Замените <Имя-исходящего-интерфейса> на имя вашего исходящего интерфейса.
%%

Спасибо за предоставленную информацию. Похоже, что у вас есть два маршрутизатора (ISP и RTR-R), каждый из которых имеет интерфейс ens33, подключенный к одной и той же сети 5.5.5.0/24.

Если вы хотите, чтобы RTR-R мог подключаться к интернету через ISP, вам нужно добавить маршрут по умолчанию на RTR-R, указывающий на IP-адрес ISP в этой сети. В вашем случае это будет 5.5.5.1. Вы можете сделать это с помощью следующей команды на RTR-R:

sudo ip route add default via 5.5.5.1

После этого попробуйте выполнить команду ping 8.8.8.8 снова на RTR-R и посмотрите, есть ли у вас доступ в Интернет.
Таким образом, команда для настройки PAT будет выглядеть следующим образом:

sudo iptables -t nat -A POSTROUTING -o ens33 -s 192.168.200.0/24 -j MASQUERADE



Настройка GRE-туннеля: GRE (Generic Routing Encapsulation) - это протокол туннелирования, который может инкапсулировать широкий диапазон типов сетевых пакетов внутри транспортного протокола IP. В вашем случае, вы можете использовать GRE для создания виртуального приватного соединения между RTR-L и RTR-R.

На обоих маршрутизаторах выполните следующие команды:

# Создайте туннельный интерфейс
sudo ip tunnel add Tunnel1 mode gre local <Локальный-IP> remote <Удаленный-IP> ttl 255

# Включите туннельный интерфейс
sudo ip link set Tunnel1 up



Настройка OSPF: Для настройки OSPF вам потребуется установить пакет quagga. Quagga - это программное обеспечение для маршрутизации, которое поддерживает различные протоколы маршрутизации, включая OSPF.

# Установите quagga
sudo apt-get install quagga

После установки quagga, вы можете настроить OSPF, редактируя файлы конфигурации в /etc/quagga/. Вам потребуется настроить файл ospfd.conf для каждого маршрутизатора, указав сети, которые должны быть объявлены, и параметры OSPF.

Пример файла ospfd.conf может выглядеть так:

! /etc/quagga/ospfd.conf
!
router ospf
 network 192.168.1.0/24 area 0.0.0.0
 network 192.168.2.0/24 area 0.0.0.0
!
line vty
!

В этом примере, две сети (192.168.1.0/24 и 192.168.2.0/24) объявлены в OSPF. Вы должны заменить эти сети на те, которые вы хотите объявить в вашей сети.

После настройки ospfd.conf, вы можете запустить OSPF с помощью следующей команды:

sudo systemctl start ospfd

Это запустит OSPF и начнет динамическую маршрутизацию между объявленными сетями. 

# Настройка на RTR-L
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o ens33 -s 192.168.200.0/24 -j MASQUERADE
iptables -t nat -A PREROUTING -d 4.4.4.100 -p tcp --dport 80 -j DNAT --to-destination 192.168.200.200:80
iptables -t nat -A PREROUTING -d 4.4.4.100 -p tcp --dport 443 -j DNAT --to-destination 192.168.200.200:443

# Настройка на RTR-R
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o ens33 -s 172.16.100.0/24 -j MASQUERADE
iptables -t nat -A PREROUTING -d 172.16.100.254 -p tcp --dport 80 -j DNAT --to-destination 192.168.200.200:80
iptables -t nat -A PREROUTING -d 172.16.100.254 -p tcp --dport 443 -j DNAT --to-destination 192.168.200.200:443


