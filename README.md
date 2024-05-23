ЗАДАНИЕ 1 МОДУЛЬ 1
Пункт 1.
А)
BR-R (HQ-R)
configure - вход в режим конфигурации
hostname BR-R - в режиме конфигурации меняет имя устройства
exit
comm - всегда после выхода писать чтоб сохранить
confirm - подтвердить после comm
SRV
hostnamectl set-hostname HQ-SRV
exec bash - сразу применяет настройки 
——
CLI (BR-SRV)
su - зайти от имени суперпользователя
hostnamectl set-hostname HQ-CLI
exec bash
(ISP не меняем)


Б)
CLI
Параметр соединение
Параметры IPv4 - вручную
адрес 3.3.3.2 маска сети 30 шлюз 3.3.3.1
Серверы DNS 8.8.8.8
терминал:
ip a - показывает все ip на устройстве 
ping 3.3.3.1
ping ya.ru
——
HQ-R
configure
interface gigabitethernet 1/0/1 - вход в интерфейс который смотрит в сторону ISP
ip address 1.1.1.2/30  - (в интерфейсе) меняет ip
ip firewall disable - (в интерфейсе) отключает фаерволл
ex
comm
confirm
ping 1.1.1.1
——

configure
interface gigabitethernet 1/0/2
ip address 192.168.100.1/26
ip firewall disable
ex
comm
confirm
——
BR-R
configure
interface gigabitethernet 1/0/1
ip address 2.2.2.2/30
ip firewall disable
exit
interface gigabitethernet 1/0/2
ip address 172.16.100.1/28
ip firewall disable
ex
comm
confirm - подтвердить после comm
ping 2.2.2.1
——
BR-SRV
(маска) echo 172.16.100.2/28 > /etc/net/ifaces/ens192/ipv4address
(шлюз) echo via default 172.16.100.1 > /etc/net/ifaces/ens192/ipv4route
(днс сервер) echo nameserver 192.168.100.2 > /etc/net/ifaces/ens192/resolv.conf
vim /etc/net/ifaces/ens192/options
ip a - показывает все ip на устройстве 
systemctl restart network
ip a
ping 172.16.100.1

ЗАДАНИЕ 2 МОДУЛЬ 1
HQ-R BR-R
configure
object-group network LOCAL_NET
ip address-range 192.168.100.1-192.168.100.62 для BR-R 172.16.100.1-172.16.100.1
exit
object-group network PUBLIC_POOL
ip address 1.1.1.2
exit
nat source
pool TRANSLATE_ADDRESS
ip address 1.1.1.2
exit
ruleset SNAT
to interface gigabitethernet 1/0/1
rule 1
match source-address LOCAL_NET
action source-nat pool TRANSLATE_ADDRESS
enable
exit
ip route 0.0.0.0/0 1.1.1.1
exit
comm
confirm
ping 8.8.8.8
ping 8.8.8.8 source ip 192.168.100.1
show ip nat translations





Динамическая маршрутизация OSPF
configure

router ospf
router-id 1.1.1.1
area 1.1.1.1

network 192.168.1.0/26
enable

exit
enable
exit

Настройка туннеля
configure
tunnel gre 10
ip firewall disable

local address 1.1.1.2
remote address 2.2.2.2
ip address 10.10.10.1/30
mtu 1426
ttl 18
enable

exit
comm
confirm
ЗАДАНИЕ 3 МОДУЛЬ 1
Настройка DHCP

configure - вход в настройки роутера
ip dhcp-server - активация dhcp сервера
ip dhcp-server pool LAN_HQ
network <указываем сеть HQ>
address-range <пишем начало пула> - <пишем конец пула>
excluded-address-range <вписываем ip-шник шлюза>
excluded-address-range <вписываем ip-шник который хотим зарезервировать>
address <прописываем IP-шник для присваивания HQ-SRV> mac-address <Прописываем mac адрес HQ-SRV>
default-router <прописываем IP шлюз HQ-R>
dns-server <IP-шник dns сервера HQ-SRV>
exit
exit
comm
confirm
ЗАДАНИЕ 4 МОДУЛЬ 1
CLI
Меню→центр управления→центр управления системой→вводим пароль→Локальные учетные записи→Вбиваем имя учётной записи→Создать→в комментарии написать имя учётки→дальше ниже вводим пароль(как в задании)→Применить.
HQ-SRV(BR-SRV-2 учетки)
useradd (имя) -с “(имя)”
passwd (имя которое вводили при создании)
(Enter после вводим пароль)
usermod -aG wheel (имя которое вводили при создании) - Даём админку пользователю
cat /etc/paswd - проверка создания пользователя в самом низу ваш пользователь
——
HQ-R
configure
username admin
password (пароль по заданию)
exit
exit
comm
confirm
show users accounts - проверка 
при 
——
BR-R(2-учётки и HQ-R)
configure
username (имя)
password (пароль)
privilege 15
exit
exit
commit
confirm


ЗАДАНИЕ 5 МОДУЛЬ 1
CLI
su
apt-get update
apt-get install iperf3
iperf3 -c 3.3.3.1
iperf3 -R -c 3.3.3.1
ЗАДАНИЕ 6 МОДУЛЬ 1
HQ-R
configure
archive
type local
count-backup 30
time-period 1440
by-commitexit
comm
confirm
dir flash:backup/

BR-R
configure
archive
type local
count-backup 30
time-period 1440
by-commitexit
comm
confirm
dir flash:backup/

ЗАДАНИЕ 7 МОДУЛЬ 1
apt-get install openssh-server
cd /etc/openssh (раскомментировать #Port 22 и заменить порт на 2222)
systemctl restart sshd
Запускаем PTTY ввоодим ip и порт и пытаемся подключится для проверки

ЗАДАНИЕ 8 МОДУЛЬ 1
cd /etc/openssh
пишем в свободной строке AllowUsers login@ip …(Перечисляем пользователей которые должны подключаться(Все кроме Cli))
systemctl restart sshd







Модуль 2 
Задание 1
HQ-SRV 
vim /etc/bind/options.conf 
Жмем ‘’insert’’
Меняем в строке айпи на это : listen-on {  any; };
				listen-on-v6 {  any; };
в коментарии forward only удаляем // и меняем на forward first;
		forwarders { 77.88.8.8; }
закоментить строчку include «/ets/bind/resolvconf-options.conf»;
allow-query {any; }; нажимаем ‘’esc’’ сохраняем командой wq 
перезапускаем службу systemctl restart bind 
создаем зону hq.work и branch work 
поправляем файл 
заходим : vim  /etc/bind/local.conf
Создаем зоны по заданию :
Zone «hq.work» {
                                Type master;
		   File «hq.db»;
};
Прописываем зону branch.work 
Копируем , что писали выше жмем ‘’esc’’  нажимаем ‘’v’’  выделяем жмем ‘’y’’ ставим курсор куда хотим вставить  жмем ‘’esc’’ ‘’p’’
Zone « branch.work » {
                                Type master;
		   File «br.db»;
};





Создаем обратную зону 

Zone «ip адресс задом наперед  in-addr.arpa» {
                                Type master;
		   File «revershq.db»;
};

Zone «ip адресс задом наперед  in-addr.arpa» {
                                Type master;
		   File «reversbr.db»;
};
Создали две зоны  прямые и обратные , выходим командой wq
cp /etc/bind/zone/localdomain  /etc/bind/zone/hq.db
cp /etc/bind/zone/localdomain  /etc/bind/zone/br.db
cp /etc/bind/zone/127 /etc/bind/zone/revershq.db
cp /etc/bind/zone/127 /etc/bind/zone/reversbr.db
cоздадим права на файлы 
chown root:named /etc/bind/zone/hq.db
chown root:named /etc/bind/zone/br.db
chown root:named /etc/bind/zone/reversbr.db
chown root:named /etc/bind/zone/revershq.db
задали необходимые права что бы бинд читал данные 
далее редактируем каждый файл по заданию
vim /etc/bind/zone/hq.db ентыр
меняем localhost. Root. Localhost на  hq.work. root. hq.work.
меняем строчку IN   NS  localhot на  IN   NS  hq.work. 
самую нижня строчка должна выглядеть так:
hq-r     IN    A   //указываем ip адрэс в видосе :// 192.168.100.1 //хуй знает правильно или нет//
 hq-srv  IN    A   //указываем ip адрэс в видосе :// 192.168.100.2 //хуй знает правильно или нет//
заполнили прямые зоны с буковкой а , выходим и сохр. Командой wq
Cоздаем обратную зону  на br.db
vim /etc/bind/zone/br.db ентыр
меняем localhost. Root. Localhost на  branch.work. root. branch.work.
меняем строчку IN   NS  localhot на  IN   NS  branch.work.
самую нижня строчка должна выглядеть так:
br-r     IN    A   //указываем ip адрэс в видосе :// 172.16.100.1 //хуй знает правильно или нет//
br-srv     IN    A   //указываем ip адрэс в видосе :// 172.16.100.2 //хуй знает правильно или нет//
выходим и сохр. Командой wq
правим для обратной зоны 
vim /etc/bind/zone/revershq.db ентыр
меняем localhost. Root. Localhost на  hq.work. root. hq.work.
меняем строчку IN   NS  localhot на  IN   NS  hq.work. 
// вначале пишется последний актет// 1  IN  PTR hq-r.hq.work
// вначале пишется последний актет// 2  IN  PTR hq-srv.hq.work
выходим и сохр. Командой wq
vim /etc/bind/zone/reversbr.db ентыр
меняем localhost. Root. Localhost на  branch.work. root. branch.work.
IN    NS  branch.work
// вначале пишется последний актет//   IN    PTR  br-r.branch.work
Последню строчку удаляем 
выходим и сохр. Командой wq
проверяем кфг  named-checkconf -z
если ошибок нету , перезапускаем ДНС , если все же есть нам пизда
systemctl restart bind
проверяем статус 
systemctl status bind 
проверяем : host hq-r.hq.work




https://www.youtube.com/watch?v=D9nF4YB5NRE 1.1
https://www.youtube.com/watch?v=56o8LY2fl5I 1.2
https://www.youtube.com/watch?v=6HOqEDV4fXM 1.3
https://www.youtube.com/watch?v=QEVgk0ZHtTA 1.4
https://www.youtube.com/watch?v=MVCkT8nM9Bc 1.5
https://www.youtube.com/watch?v=dMWsAg8fI90 1.6



















