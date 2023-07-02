# Домашнее задание к лекции "Физический уровень PostgreSQL"
## Дата: 22.06.2023
## Тема: Установка и настройка PostgreSQL
## Вычислительные ресурсы
Ноутбук Lenovo T14G1 (i5, 4 core, 32 Gb, 1 Tb), ОС W10Pro, WMware WksPro 17
ВМ (2 core, 8 Gb, 30 Gb), ОС AstraLinux 1.7.3 mode 0, ПО Тантор (СУБД 15, Платформа 2.01)
## Описание/Пошаговая инструкция выполнения домашнего задания:
_создайте виртуальную машину c Ubuntu 20.04/22.04 LTS в GCE/ЯО/Virtual Box/докере
поставьте на нее PostgreSQL 15 через sudo apt
проверьте что кластер запущен через sudo -u postgres pg_lsclusters
зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым
postgres=# create table test(c1 text);
postgres=# insert into test values('1');
\q
остановите postgres например через sudo -u postgres pg_ctlcluster 15 main stop
создайте новый диск к ВМ размером 10GB
добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux
перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)
сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
перенесите содержимое /var/lib/postgres/15 в /mnt/data - mv /var/lib/postgresql/15/mnt/data
попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
напишите получилось или нет и почему
задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/15/main который надо поменять и поменяйте его
напишите что и почему поменяли
попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
напишите получилось или нет и почему
зайдите через через psql и проверьте содержимое ранее созданной таблицы
задание со звездочкой *: не удаляя существующий инстанс ВМ сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось._

## Выполнение
_Создаем две виртуальные машины. На обоих ВМ производим установку СУБД PostgreSQL без инициализации. Первая ВМ будет использоваться для тестирования переноса каталога данных на новый диск. После завершения тетстирования диск от первой ВМ будет подключен ко второй._
### 1 и 2
```
root@alse174cmdfdt1:~# apt update
Сущ:1 https://download.astralinux.ru/astra/stable/1.7_x86-64/repository-base 1.7_x86-64 InRelease
Сущ:2 https://download.astralinux.ru/astra/stable/1.7_x86-64/repository-extended 1.7_x86-64 InRelease
Чтение списков пакетов… Готово
...
root@alse174cmdfdt1:~# wget --quiet -O - https://public.tantorlabs.ru/tantorlabs.ru.asc | apt-key add -
OK
...
root@alse174cmdfdt1:~# echo "deb [arch=amd64] https://<USER_LOGIN>:<USER_PASSWORD>@nexus.tantorlabs.ru/repository/astra-smolensk-1.7 smolensk main" > /etc/apt/sources.list.d/tantorlabs.list
root@alse174cmdfdt1:~# apt update
Пол:1 https://nexus.tantorlabs.ru/repository/astra-smolensk-1.7 smolensk InRelease [1 554 B]
Сущ:2 https://download.astralinux.ru/astra/stable/1.7_x86-64/repository-base 1.7_x86-64 InRelease
Сущ:3 https://download.astralinux.ru/astra/stable/1.7_x86-64/repository-extended 1.7_x86-64 InRelease
...
root@alse174cmdfdt1:~# apt-get install tantor-se-server-15 -y
Чтение списков пакетов… Готово
Построение дерева зависимостей
Чтение информации о состоянии… Готово
Следующие НОВЫЕ пакеты будут установлены:
  tantor-se-server-15
...
+ chown postgres:postgres /var/lib/postgresql/tantor-se-15/data
+ chmod 700 /var/lib/postgresql/tantor-se-15/data
+ echo ---------------------------------------------
---------------------------------------------
+ set +vx
```
_Пооверяем наличие необходимых настроек переменных среды для работы СУБД. Конфигурируем отсутствующие переменные._

### 1
```
root@alse174cmdfdt1:~# su - postgres
postgres@alse174cmdfdt1:~$ env
SHELL=/bin/bash
PWD=/var/lib/postgresql
LOGNAME=postgres
HOME=/var/lib/postgresql
LANG=ru_RU.UTF-8
TERM=xterm
USER=postgres
SHLVL=1
PATH=/opt/tantor/db/15/bin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
MAIL=/var/mail/postgres
_=/usr/bin/env
postgres@alse174cmdfdt1:~$ nano .bash_profile
postgres@alse174cmdfdt1:~$ cat .bash_profile
export PATH=/opt/tantor/db/15/bin:$PATH
export LANGUAGE=ru_RU.UTF-8
export LC_ALL=ru_RU.UTF-8
export PGDATA=/var/lib/postgresql/tantor-se-15/data
postgres@alse174cmdfdt1:~$ env
SHELL=/bin/bash
PWD=/var/lib/postgresql
LOGNAME=postgres
HOME=/var/lib/postgresql
LANG=ru_RU.UTF-8
TERM=xterm
USER=postgres
SHLVL=1
PATH=/opt/tantor/db/15/bin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
MAIL=/var/mail/postgres
_=/usr/bin/env
postgres@alse174cmdfdt1:~$ выход
root@alse174cmdfdt1:~# su - postgres
postgres@alse174cmdfdt1:~$ env
SHELL=/bin/bash
LANGUAGE=ru_RU.UTF-8
PWD=/var/lib/postgresql
LOGNAME=postgres
HOME=/var/lib/postgresql
LANG=ru_RU.UTF-8
TERM=xterm
USER=postgres
SHLVL=1
PGDATA=/var/lib/postgresql/tantor-se-15/data
LC_ALL=ru_RU.UTF-8
PATH=/opt/tantor/db/15/bin:/opt/tantor/db/15/bin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
MAIL=/var/mail/postgres
_=/usr/bin/env
...
```
_Инициализируем СУБД. Проверяем работу СУБД. Создаем и наполняем таблицу._
### 1
```
postgres@alse174cmdfdt1:~$ initdb -D /var/lib/postgresql/tantor-se-15/data --no-instructions 2> /dev/null
Файлы, относящиеся к этой СУБД, будут принадлежать пользователю "postgres".
От его имени также будет запускаться процесс сервера.

Кластер баз данных будет инициализирован с локалью "ru_RU.UTF-8".
Кодировка БД по умолчанию, выбранная в соответствии с настройками: "UTF8".
Выбрана конфигурация текстового поиска по умолчанию "russian".

Контроль целостности страниц данных отключён.

исправление прав для существующего каталога /var/lib/postgresql/tantor-se-15/data... ок
создание подкаталогов... ок
выбирается реализация динамической разделяемой памяти... posix
выбирается значение max_connections по умолчанию... 100
выбирается значение shared_buffers по умолчанию... 128MB
выбирается часовой пояс по умолчанию... Europe/Moscow
создание конфигурационных файлов... ок
выполняется подготовительный скрипт... ок
выполняется заключительная инициализация... ок
сохранение данных на диске... ок

postgres@alse174cmdfdt1:~$ выход
root@alse174cmdfdt1:~# systemctl status tantor-se-server-15.service
● tantor-se-server-15.service - Tantor Special Edition database server 15
   Loaded: loaded (/lib/systemd/system/tantor-se-server-15.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: https://tantorlabs.ru/docs/
root@alse174cmdfdt1:~# systemctl start tantor-se-server-15.service
root@alse174cmdfdt1:~# systemctl status tantor-se-server-15.service
● tantor-se-server-15.service - Tantor Special Edition database server 15
   Loaded: loaded (/lib/systemd/system/tantor-se-server-15.service; disabled; vendor preset: enabled)
   Active: active (running) since Sun 2023-07-02 22:21:45 MSK; 3s ago
     Docs: https://tantorlabs.ru/docs/
  Process: 1209 ExecStartPre=/opt/tantor/db/15/bin/postgresql-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
  Process: 1211 ExecStart=/opt/tantor/db/15/bin/pg_ctl start -D ${PGDATA} -s -w -t ${PGSTARTTIMEOUT} (code=exited, status=0/SU
 Main PID: 1213 (postgres)
    Tasks: 6 (limit: 4596)
   Memory: 17.4M
      CPU: 74ms
   CGroup: /system.slice/tantor-se-server-15.service
           ├─1213 /opt/tantor/db/15/bin/postgres -D /var/lib/postgresql/tantor-se-15/data
           ├─1214 postgres: checkpointer
           ├─1215 postgres: background writer
           ├─1217 postgres: walwriter
           ├─1218 postgres: autovacuum launcher
           └─1219 postgres: logical replication launcher

июл 02 22:21:45 alse174cmdfdt1 systemd[1]: Starting Tantor Special Edition database server 15...
июл 02 22:21:45 alse174cmdfdt1 pg_ctl[1211]: 2023-07-02 22:21:45.633 MSK [1213] СООБЩЕНИЕ:  запускается PostgreSQL 15.2 on x86
июл 02 22:21:45 alse174cmdfdt1 pg_ctl[1211]: 2023-07-02 22:21:45.634 MSK [1213] СООБЩЕНИЕ:  для приёма подключений по адресу I
июл 02 22:21:45 alse174cmdfdt1 pg_ctl[1211]: 2023-07-02 22:21:45.634 MSK [1213] СООБЩЕНИЕ:  для приёма подключений по адресу I
июл 02 22:21:45 alse174cmdfdt1 pg_ctl[1211]: 2023-07-02 22:21:45.635 MSK [1213] СООБЩЕНИЕ:  для приёма подключений открыт Unix
июл 02 22:21:45 alse174cmdfdt1 pg_ctl[1211]: 2023-07-02 22:21:45.640 MSK [1216] СООБЩЕНИЕ:  система БД была выключена: 2023-07
июл 02 22:21:45 alse174cmdfdt1 pg_ctl[1211]: 2023-07-02 22:21:45.646 MSK [1213] СООБЩЕНИЕ:  система БД готова принимать подклю
июл 02 22:21:45 alse174cmdfdt1 systemd[1]: Started Tantor Special Edition database server 15.
root@alse174cmdfdt1:~# systemctl enable tantor-se-server-15.service
Created symlink /etc/systemd/system/multi-user.target.wants/tantor-se-server-15.service → /lib/systemd/system/tantor-se-server-15.service.
root@alse174cmdfdt1:~# su - postgres
postgres@alse174cmdfdt1:~$ pg_ctl status
pg_ctl: сервер работает (PID: 1213)
/opt/tantor/db/15/bin/postgres "-D" "/var/lib/postgresql/tantor-se-15/data"
postgres@alse174cmdfdt1:~$ psql
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# create table test(c1 text);
CREATE TABLE
postgres=# insert into test values('1');
INSERT 0 1
postgres=# select c1 from test;
 c1
----
 1
(1 строка)

postgres=# \q
```
_Получаем информацию о расположении данных и файла конфигурации. Останавливаем СУБД. Добавляем в конфигурацию первой ВМ диск 10 Гб._
### 1
```
postgres@alse174cmdfdt1:~$ pg_ctl status
pg_ctl: сервер работает (PID: 913)
/opt/tantor/db/15/bin/postgres "-D" "/var/lib/postgresql/tantor-se-15/data"
postgres@alse174cmdfdt1:~$ psql -c "SHOW config_file;"
                      config_file
-------------------------------------------------------
 /var/lib/postgresql/tantor-se-15/data/postgresql.conf
(1 строка)

postgres@alse174cmdfdt1:~$ psql -c "SHOW data_directory;"
            data_directory
---------------------------------------
 /var/lib/postgresql/tantor-se-15/data
(1 строка)

postgres@alse174cmdfdt1:~$ pg_ctl stop
ожидание завершения работы сервера.... готово
сервер остановлен
postgres@alse174cmdfdt1:~$ выход
root@alse174cmdfdt1:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk
├─sda1   8:1    0   19G  0 part /
├─sda2   8:2    0    1K  0 part
└─sda5   8:5    0  975M  0 part [SWAP]
sr0     11:0    1 1024M  0 rom
root@alse174cmdfdt1:~# fdisk -l
Диск /dev/sda: 20 GiB, 21474836480 байт, 41943040 секторов
Disk model: VMware Virtual S
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт
Тип метки диска: dos
Идентификатор диска: 0xe55e08f1

Устр-во    Загрузочный   начало    Конец  Секторы Размер Идентификатор Тип
/dev/sda1  *               2048 39942143 39940096    19G            83 Linux
/dev/sda2              39944190 41940991  1996802   975M             5 Расширенный
/dev/sda5              39944192 41940991  1996800   975M            82 Linux своп / Solaris
```
_Не останавливая работу ВМ сканируем доступные (вновь появившиеся) диски. Устанавливаем на новый диск загрузчик, выполняем разметку файловой системы. Получаем идентификатор диска и вносим изменения в файл автомонтирования дисков. Проверяем корректность внесенных изменений._
### 1
```
root@alse174cmdfdt1:~# echo "- - -" | tee /sys/class/scsi_host/host*/scan
- - -
root@alse174cmdfdt1:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk
├─sda1   8:1    0   19G  0 part /
├─sda2   8:2    0    1K  0 part
└─sda5   8:5    0  975M  0 part [SWAP]
sdb      8:16   0   10G  0 disk
sr0     11:0    1 1024M  0 rom
root@alse174cmdfdt1:~# fdisk -l
Диск /dev/sda: 20 GiB, 21474836480 байт, 41943040 секторов
Disk model: VMware Virtual S
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт
Тип метки диска: dos
Идентификатор диска: 0xe55e08f1

Устр-во    Загрузочный   начало    Конец  Секторы Размер Идентификатор Тип
/dev/sda1  *               2048 39942143 39940096    19G            83 Linux
/dev/sda2              39944190 41940991  1996802   975M             5 Расширенный
/dev/sda5              39944192 41940991  1996800   975M            82 Linux своп / Solaris


Диск /dev/sdb: 10 GiB, 10737418240 байт, 20971520 секторов
Disk model: VMware Virtual S
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт
root@alse174cmdfdt1:~# fdisk /dev/sdb

Добро пожаловать в fdisk (util-linux 2.33.1).
Изменения останутся только в памяти до тех пор, пока вы не решите записать их.
Будьте внимательны, используя команду write.

Устройство не содержит стандартной таблицы разделов.
Создана новая метка DOS с идентификатором 0x4dba2b90.

Команда (m для справки): g
Created a new GPT disklabel (GUID: AF123ADC-8785-4E44-B27A-4A9E288547CA).

Команда (m для справки): F
Неразмеченное место /dev/sdb: 10 GiB, 10736352768 байт, 20969439 секторов
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт

начало    Конец  Секторы Размер
  2048 20971486 20969439    10G

Команда (m для справки): n
Номер раздела (1-128, default 1):
Первый сектор (2048-20971486, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-20971486, default 20971486):

Создан новый раздел 1 с типом 'Linux filesystem' и размером 10 GiB.

Команда (m для справки): w
Таблица разделов была изменена.
Вызывается ioctl() для перечитывания таблицы разделов.
Синхронизируются диски.

root@alse174cmdfdt1:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk
├─sda1   8:1    0   19G  0 part /
├─sda2   8:2    0    1K  0 part
└─sda5   8:5    0  975M  0 part [SWAP]
sdb      8:16   0   10G  0 disk
└─sdb1   8:17   0   10G  0 part
sr0     11:0    1 1024M  0 rom
root@alse174cmdfdt1:~# mkfs.ext4 /dev/sdb1
mke2fs 1.44.5 (15-Dec-2018)
Creating filesystem with 2621179 4k blocks and 655360 inodes
Filesystem UUID: d10cc72b-56a6-4b3c-acb3-f989ac4c2c69
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

root@alse174cmdfdt1:~# blkid
/dev/sda1: UUID="b75e9c14-23aa-46ad-a399-a5c231c5662a" TYPE="ext4" PARTUUID="e55e08f1-01"
/dev/sda5: UUID="5c5da6bf-46fa-4670-ba49-b4d035c462ad" TYPE="swap" PARTUUID="e55e08f1-05"
/dev/sdb1: UUID="d10cc72b-56a6-4b3c-acb3-f989ac4c2c69" TYPE="ext4" PARTUUID="63a3ac21-a02d-9146-b056-1c2f6e3244fe"
root@alse174cmdfdt1:~# mkdir /mnt/data
root@alse174cmdfdt1:~# cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda1 during installation
UUID=b75e9c14-23aa-46ad-a399-a5c231c5662a /               ext4    errors=remount-ro 0       1
# swap was on /dev/sda5 during installation
UUID=5c5da6bf-46fa-4670-ba49-b4d035c462ad none            swap    sw              0       0
/dev/sr0        /media/cdrom0   udf,iso9660 user,noauto     0       0
root@alse174cmdfdt1:~# echo "UUID=d10cc72b-56a6-4b3c-acb3-f989ac4c2c69 /mnt/data ext4 errors=remount-ro 0 2" >> /etc/fstab
root@alse174cmdfdt1:~# cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda1 during installation
UUID=b75e9c14-23aa-46ad-a399-a5c231c5662a /               ext4    errors=remount-ro 0       1
# swap was on /dev/sda5 during installation
UUID=5c5da6bf-46fa-4670-ba49-b4d035c462ad none            swap    sw              0       0
/dev/sr0        /media/cdrom0   udf,iso9660 user,noauto     0       0
UUID=d10cc72b-56a6-4b3c-acb3-f989ac4c2c69 /mnt/data ext4 errors=remount-ro 0 2
root@alse174cmdfdt1:~# mount |grep ext4
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro)
root@alse174cmdfdt1:~# mount /mnt/data
root@alse174cmdfdt1:~# mount |grep ext4
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro)
/dev/sdb1 on /mnt/data type ext4 (rw,relatime,errors=remount-ro)
```
_Перезагружаем ВМ для проверки автомонтирования нового каталога данных на подключенном диске._
### 1
```
root@alse174cmdfdt1:~# shutdown -r now
Using username "daylight".
daylight@192.168.186.171's password:
Send automatic password
Last login: Sun Jul  2 20:57:50 2023 from 192.168.186.1
daylight@alse174cmdfdt1:~$ sudo su -
root@alse174cmdfdt1:~# mount |grep ext4
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro)
/dev/sdb1 on /mnt/data type ext4 (rw,relatime,errors=remount-ro)
```
_Назначаем новому каталогу данных необходимые атрибуты и владельца. Проверяем настройки. Верифицируем подключенный диск._
### 1
```
root@alse174cmdfdt1:~# chown -R postgres:postgres /mnt/data
root@alse174cmdfdt1:~# chmod 0750 /mnt/data
root@alse174cmdfdt1:~# ls -lha /mnt/
итого 12K
drwxr-xr-x  3 root     root     4,0K июл  2 23:16 .
drwxr-xr-x 20 root     root     4,0K июл  2 23:06 ..
drwxr-x--- 20 postgres postgres 4,0K июл  2 23:28 data
root@alse174cmdfdt1:~# df -h
Файловая система Размер Использовано  Дост Использовано% Cмонтировано в
udev               1,9G            0  1,9G            0% /dev
tmpfs              390M         6,0M  384M            2% /run
/dev/sda1           19G         2,6G   16G           15% /
tmpfs              2,0G         1,1M  1,9G            1% /dev/shm
tmpfs              5,0M            0  5,0M            0% /run/lock
/dev/sdb1          9,8G          24K  9,3G            1% /mnt/data
root@alse174cmdfdt1:~#
_Останавливаем СУБД (автозапуск после перезагрузки ВМ). Выполняем необходимые настройки по изменению места расположения данных. Копируем данные на новое место. Запускаем СУБД. Проверяем тестовую таблицу. Выключаем первую ВМ._
### 1
```
postgres@alse174cmdfdt1:~$ pg_ctl stop
2023-07-02 22:44:55.782 MSK [995] СООБЩЕНИЕ:  получен запрос на быстрое выключение
ожидание завершения работы сервера....2023-07-02 22:44:55.785 MSK [995] СООБЩЕНИЕ:  прерывание всех активных транзакций
2023-07-02 22:44:55.795 MSK [995] СООБЩЕНИЕ:  фоновый процесс "logical replication launcher" (PID 1001) завершился с кодом выхода 1
2023-07-02 22:44:55.798 MSK [996] СООБЩЕНИЕ:  выключение
2023-07-02 22:44:55.801 MSK [996] СООБЩЕНИЕ:  начата контрольная точка: shutdown immediate
2023-07-02 22:44:55.817 MSK [996] СООБЩЕНИЕ:  контрольная точка завершена: записано буферов: 3 (0.0%); добавлено файлов WAL 0, удалено: 0, переработано: 0; запись=0.005 сек., синхр.=0.002 сек., всего=0.019 сек.; синхронизировано_файлов=2, самая_долгая_синхр.=0.001 сек., средняя=0.001 сек.; расстояние=0 kB, ожидалось=0 kB
2023-07-02 22:44:55.823 MSK [995] СООБЩЕНИЕ:  система БД выключена
 готово
сервер остановлен
postgres@alse174cmdfdt1:~$ pg_ctl status
pg_ctl: сервер не работает
postgres@alse174cmdfdt1:~$ cp -pr /var/lib/postgresql/tantor-se-15/data/* /mnt/data/
postgres@alse174cmdfdt1:~$ nano /var/lib/postgresql/tantor-se-15/data/postgresql.conf
postgres@alse174cmdfdt1:~$ cat /var/lib/postgresql/tantor-se-15/data/postgresql.conf|grep 'data_directory ='
#data_directory = 'ConfigDir'           # use data in another directory
data_directory = '/mnt/data'
root@alse174cmdfdt1:~# nano /lib/systemd/system/tantor-se-15.service
root@alse174cmdfdt1:~# cat /lib/systemd/system/tantor-se-15.service
Environment=PGDATA=/mnt/data
root@alse174cmdfdt1:~# su - postgres
postgres@alse174cmdfdt1:~$ pg_ctl start
ожидание запуска сервера....2023-07-02 22:55:55.306 MSK [1059] СООБЩЕНИЕ:  запускается PostgreSQL 15.2 on x86_64-pc-linux-gnu, compiled by gcc (AstraLinuxSE 8.3.0-6) 8.3.0, 64-bit
2023-07-02 22:55:55.307 MSK [1059] СООБЩЕНИЕ:  для приёма подключений по адресу IPv4 "127.0.0.1" открыт порт 5432
2023-07-02 22:55:55.307 MSK [1059] СООБЩЕНИЕ:  для приёма подключений по адресу IPv6 "::1" открыт порт 5432
2023-07-02 22:55:55.309 MSK [1059] СООБЩЕНИЕ:  для приёма подключений открыт Unix-сокет "/var/run/postgresql/.s.PGSQL.5432"
2023-07-02 22:55:55.317 MSK [1062] СООБЩЕНИЕ:  система БД была выключена: 2023-07-02 22:44:55 MSK
2023-07-02 22:55:55.324 MSK [1059] СООБЩЕНИЕ:  система БД готова принимать подключения
postgres@alse174cmdfdt1:~$ psql
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# select c1 from test;
 c1
----
 1
(1 строка)

postgres=# \q
postgres@alse174cmdfdt1:~$
postgres@alse174cmdfdt1:~$ выход
root@alse174cmdfdt1:~# shutdown -h now
```
_Идентифицируем место расположения диска с данными СУБД в каталоге первой ВМ._
C:\Users\daylight\Documents\Virtual Machines\Clone of ALSE174CMDFDT1\ALSE174CMDFDT1-cl1.vmdk
_Выполняем подключение диска первой ВМ ко второй ВМ. Конфигурируем точку монтирования, параметры СУБД (аналогично первой ВМ). Запускаем СУБД. Проверяем тестовую таблицу._
### 2
```
daylight@alse174cmdfdt2:~$ sudo su -
root@alse174cmdfdt2:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk
├─sda1   8:1    0   19G  0 part /
├─sda2   8:2    0    1K  0 part
└─sda5   8:5    0  975M  0 part [SWAP]
sr0     11:0    1 1024M  0 rom
root@alse174cmdfdt2:~# fdisk -l
Диск /dev/sda: 20 GiB, 21474836480 байт, 41943040 секторов
Disk model: VMware Virtual S
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт
Тип метки диска: dos
Идентификатор диска: 0xe55e08f1

Устр-во    Загрузочный   начало    Конец  Секторы Размер Идентификатор Тип
/dev/sda1  *               2048 39942143 39940096    19G            83 Linux
/dev/sda2              39944190 41940991  1996802   975M             5 Расширенный
/dev/sda5              39944192 41940991  1996800   975M            82 Linux своп / Solaris
root@alse174cmdfdt2:~# echo "- - -" | tee /sys/class/scsi_host/host*/scan
- - -
root@alse174cmdfdt2:~# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk
├─sda1   8:1    0   19G  0 part /
├─sda2   8:2    0    1K  0 part
└─sda5   8:5    0  975M  0 part [SWAP]
sdb      8:16   0   10G  0 disk
└─sdb1   8:17   0   10G  0 part
sr0     11:0    1 1024M  0 rom
root@alse174cmdfdt2:~# fdisk -l
Диск /dev/sda: 20 GiB, 21474836480 байт, 41943040 секторов
Disk model: VMware Virtual S
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт
Тип метки диска: dos
Идентификатор диска: 0xe55e08f1

Устр-во    Загрузочный   начало    Конец  Секторы Размер Идентификатор Тип
/dev/sda1  *               2048 39942143 39940096    19G            83 Linux
/dev/sda2              39944190 41940991  1996802   975M             5 Расширенный
/dev/sda5              39944192 41940991  1996800   975M            82 Linux своп / Solaris


Диск /dev/sdb: 10 GiB, 10737418240 байт, 20971520 секторов
Disk model: VMware Virtual S
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт
Тип метки диска: gpt
Идентификатор диска: AF123ADC-8785-4E44-B27A-4A9E288547CA

Устр-во    начало    Конец  Секторы Размер Тип
/dev/sdb1    2048 20971486 20969439    10G Файловая система Linux
root@alse174cmdfdt2:~# blkid
/dev/sda1: UUID="b75e9c14-23aa-46ad-a399-a5c231c5662a" TYPE="ext4" PARTUUID="e55e08f1-01"
/dev/sda5: UUID="5c5da6bf-46fa-4670-ba49-b4d035c462ad" TYPE="swap" PARTUUID="e55e08f1-05"
/dev/sdb1: UUID="d10cc72b-56a6-4b3c-acb3-f989ac4c2c69" TYPE="ext4" PARTUUID="63a3ac21-a02d-9146-b056-1c2f6e3244fe"
root@alse174cmdfdt2:~# cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda1 during installation
UUID=b75e9c14-23aa-46ad-a399-a5c231c5662a /               ext4    errors=remount-ro 0       1
# swap was on /dev/sda5 during installation
UUID=5c5da6bf-46fa-4670-ba49-b4d035c462ad none            swap    sw              0       0
/dev/sr0        /media/cdrom0   udf,iso9660 user,noauto     0       0
root@alse174cmdfdt2:~# mkdir /mnt/data
root@alse174cmdfdt2:~# echo "UUID=d10cc72b-56a6-4b3c-acb3-f989ac4c2c69 /mnt/data ext4 errors=remount-ro 0 2" >> /etc/fstab
root@alse174cmdfdt2:~# cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda1 during installation
UUID=b75e9c14-23aa-46ad-a399-a5c231c5662a /               ext4    errors=remount-ro 0       1
# swap was on /dev/sda5 during installation
UUID=5c5da6bf-46fa-4670-ba49-b4d035c462ad none            swap    sw              0       0
/dev/sr0        /media/cdrom0   udf,iso9660 user,noauto     0       0
UUID=d10cc72b-56a6-4b3c-acb3-f989ac4c2c69 /mnt/data ext4 errors=remount-ro 0 2
root@alse174cmdfdt2:~# mount |grep ext4
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro)
root@alse174cmdfdt2:~# mount /mnt/data
root@alse174cmdfdt2:~# mount |grep ext4
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro)
/dev/sdb1 on /mnt/data type ext4 (rw,relatime,errors=remount-ro)
root@alse174cmdfdt2:~# df -h
Файловая система Размер Использовано  Дост Использовано% Cмонтировано в
udev               1,9G            0  1,9G            0% /dev
tmpfs              390M         6,0M  384M            2% /run
/dev/sda1           19G         2,6G   16G           15% /
tmpfs              2,0G         8,0K  2,0G            1% /dev/shm
tmpfs              5,0M            0  5,0M            0% /run/lock
/dev/sdb1          9,8G          39M  9,2G            1% /mnt/data
root@alse174cmdfdt2:~# chown -R postgres:postgres /mnt/data
root@alse174cmdfdt2:~# chmod 0750 /mnt/data
root@alse174cmdfdt2:~# ls -lha /mnt/data
итого 148K
drwxr-x--- 20 postgres postgres 4,0K июл  2 23:10 .
drwxr-xr-x  3 root     root     4,0K июл  2 23:16 ..
drwx------  5 postgres postgres 4,0K июл  2 22:20 base
drwx------  2 postgres postgres 4,0K июл  2 22:56 global
drwx------  2 postgres postgres  16K июл  2 22:26 lost+found
drwx------  2 postgres postgres 4,0K июл  2 22:20 pg_commit_ts
drwx------  2 postgres postgres 4,0K июл  2 22:20 pg_dynshmem
-rw-------  1 postgres postgres 4,7K июл  2 22:20 pg_hba.conf
-rw-------  1 postgres postgres 1,6K июл  2 22:20 pg_ident.conf
drwx------  4 postgres postgres 4,0K июл  2 23:10 pg_logical
drwx------  4 postgres postgres 4,0K июл  2 22:20 pg_multixact
drwx------  2 postgres postgres 4,0K июл  2 22:20 pg_notify
drwx------  2 postgres postgres 4,0K июл  2 22:20 pg_replslot
drwx------  2 postgres postgres 4,0K июл  2 22:20 pg_serial
drwx------  2 postgres postgres 4,0K июл  2 22:20 pg_snapshots
drwx------  2 postgres postgres 4,0K июл  2 23:10 pg_stat
drwx------  2 postgres postgres 4,0K июл  2 22:20 pg_stat_tmp
drwx------  2 postgres postgres 4,0K июл  2 22:20 pg_subtrans
drwx------  2 postgres postgres 4,0K июл  2 22:20 pg_tblspc
drwx------  2 postgres postgres 4,0K июл  2 22:20 pg_twophase
-rw-------  1 postgres postgres    3 июл  2 22:20 PG_VERSION
drwx------  3 postgres postgres 4,0K июл  2 22:20 pg_wal
drwx------  2 postgres postgres 4,0K июл  2 22:20 pg_xact
-rw-------  1 postgres postgres   88 июл  2 22:20 postgresql.auto.conf
-rw-------  1 postgres postgres  30K июл  2 22:20 postgresql.conf
-rw-------  1 postgres postgres   31 июл  2 22:55 postmaster.opts
root@alse174cmdfdt2:~# su - postgres
postgres@alse174cmdfdt2:~$ nano .bash_profile
postgres@alse174cmdfdt2:~$ cat .bash_profile
export PATH=/opt/tantor/db/15/bin:$PATH
export LANGUAGE=ru_RU.UTF-8
export LC_ALL=ru_RU.UTF-8
export PGDATA=/mnt/data
postgres@alse174cmdfdt2:~$ выход
root@alse174cmdfdt2:~# nano /lib/systemd/system/tantor-se-15.service
root@alse174cmdfdt2:~# cat /lib/systemd/system/tantor-se-15.service
Environment=PGDATA=/mnt/data
root@alse174cmdfdt2:~# su - postgres
postgres@alse174cmdfdt2:~$ pg_ctl start
ожидание запуска сервера....2023-07-02 23:28:20.537 MSK [1347] СООБЩЕНИЕ:  запускается PostgreSQL 15.2 on x86_64-pc-linux-gnu, compiled by gcc (AstraLinuxSE 8.3.0-6) 8.3.0, 64-bit
2023-07-02 23:28:20.540 MSK [1347] СООБЩЕНИЕ:  для приёма подключений по адресу IPv4 "127.0.0.1" открыт порт 5432
2023-07-02 23:28:20.541 MSK [1347] СООБЩЕНИЕ:  для приёма подключений по адресу IPv6 "::1" открыт порт 5432
2023-07-02 23:28:20.542 MSK [1347] СООБЩЕНИЕ:  для приёма подключений открыт Unix-сокет "/var/run/postgresql/.s.PGSQL.5432"
2023-07-02 23:28:20.548 MSK [1350] СООБЩЕНИЕ:  система БД была выключена: 2023-07-02 23:10:34 MSK
2023-07-02 23:28:20.562 MSK [1347] СООБЩЕНИЕ:  система БД готова принимать подключения
 готово
сервер запущен
postgres@alse174cmdfdt2:~$ psql
psql (15.2)
Введите "help", чтобы получить справку.

postgres=# select c1 from test;
 c1
----
 1
(1 строка)

postgres=# \q
postgres@alse174cmdfdt2:~$ выход
root@alse174cmdfdt2:~#
```