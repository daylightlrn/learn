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
_Создаем две виртуальные машины, на первой размещаем контейнеры, вторая будет использоваться для удаленного доступа к контейнеру с СУБД. Устанавливаем на первую ВМ пакет Docker Engine. На второй ВМ устанавливаем клиент СУБД PostgreSQL._
### 1
```
root@postgres14:~# apt update

Чтение информации о состоянии… Готово
...
```
_Создаем на первой ВМ каталог /var/lib/postgres. Готовим сеть для взаимодействия контейнеров. Загружаем и запускаем контейнер с СУБД PostgreSQL 15. Монтируем в него каталог /var/lib/postgres. Проверяем готовность контейнера и сети._

### 2
```
root@postgres14:~# mkdir -p /var/lib/postgres

root@postgres14:~# docker network create psql-net
87ccb7ef6b0cdb8b008260421733a50b97c2a30b7cda643ee920a224cb4f302f

root@postgres14:~# docker run --name psql-srv --network psql-net -e POSTGRES_PASSWORD='1!Password' -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
Unable to find image 'postgres:15' locally
15: Pulling from library/postgres
5b5fe70539cd: Pull complete
...


```