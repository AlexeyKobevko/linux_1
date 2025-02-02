## Финальная домашняя работа Алексея Кобевко

>Установить в виртуальную машину или VDS Docker, сделать два контейнера, один для Nginx, второй для MySQL, настроить совместную работу. Если вы изучаете программирование на PHP, Python или Java, используйте их в проектах для работы с БД.

#### ШАГ 1. Установака **Docker**
##### Дистрибутив Docker, доступный в официальном репозитории Ubuntu, не всегда является последней версией программы. Лучше установить последнюю версию Docker, загрузив ее из официального репозитория Docker. Для этого добавляем новый источник дистрибутива, вводим ключ GPG из репозитория Docker, чтобы убедиться, действительна ли загруженная версия, а затем устанавливаем дистрибутив.
Сначала обновляем существующий перечень пакетов:
```bash
$ sudo apt update
```
Затем устанавливаем необходимые пакеты, которые позволяют apt использовать пакеты по HTTPS:
```bash
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
```
Затем добавляем в свою систему ключ GPG официального репозитория Docker:
```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
Добавляем репозиторий Docker в список источников пакетов APT:
```bash
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
```
Затем обновим базу данных пакетов информацией о пакетах Docker из вновь добавленного репозитория:
```bash
$ sudo apt update
```
Следует убедиться, что мы устанавливаем Docker из репозитория Docker, а не из репозитория по умолчанию Ubuntu:
```bash
$ apt-cache policy docker-ce
```
Вывод получится приблизительно следующий:
```
docker-ce:
  Установлен: (отсутствует)
  Кандидат:   5:19.03.2~3-0~ubuntu-bionic
  Таблица версий:
     5:19.03.2~3-0~ubuntu-bionic 500
        500 https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
     5:19.03.1~3-0~ubuntu-bionic 500
        500 https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
```
```docker-ce``` не устанавливается, но для установки будет использован репозиторий Docker для Ubuntu 18.04 (bionic).


Далее устанавливаем **Docker**:
```bash
$ sudo apt install docker-ce
```
Теперь **Docker** установлен, демон запущен, и процесс будет запускаться при загрузке системы.  Убедимся, что процесс запущен:
```bash
$ sudo systemctl status docker
```
Вывод должен быть похож на представленный ниже, сервис должен быть запущен и активен:
```
jedi@jedi-VirtualBox:~$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2019-09-12 10:58:45 MSK; 1min 42s ago
     Docs: https://docs.docker.com
 Main PID: 17744 (dockerd)
    Tasks: 8
   CGroup: /system.slice/docker.service
           └─17744 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

#### ШАГ 2. Использование команды **Docker** без sudo

Чтобы не вводить ```sudo``` каждый раз при запуске команды docker, добавляю имя своего пользователя в группу ```docker```:
```bash
$ sudo usermod -aG docker jedi
$ su - jedi
$ id -nG
```
#### ШАГ 3. Поиск и установка образов **Nginx** и **MySQL**

Ищем образ **Nginx**
```bash
$ docker search nginx
```
Устанавливаю понравившийся образ
```bash
$ docker pull richarvey/nginx-php-fpm
$ docker run -d --name nginx-php -p 80:80 -v /var/www/html:/user/share/nginx/html/ richarvey/nginx-php-fpm
$ docker ps
```
```
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                                   NAMES
75cc25a13830        richarvey/nginx-php-fpm   "docker-php-entrypoi…"   4 minutes ago       Up 18 seconds       443/tcp, 0.0.0.0:80->80/tcp, 9000/tcp   nginx-php

```
Произвожу те же действия для **MySQL**
```bash
$ docker search mysql
$ docker pull mysql/mysql-server:latest
$ docker run --name=db -p 3306:3306 -d mysql/mysql-server:latest
```
Во время развертывания был установлен случайный пароль. Чтобы увидеть пароль, необходимо ввести команду:
```bash
$ docker logs db
```
![Db pass](./images/db_pass.png)
Захожу на сервер БД и ввожу пароль из лога
```bash
$ docker exec -it db mysql -uroot -p
```
Система приглашает к вводу команд:
![Db pass](./images/welcome.png)
Меняю пароль БД:
```bash
$ ALTER USER root@localhost IDENTIFIED BY '123';
```
![Change pass](./images/rename_pass.png)

#### ШАГ 4. Создание БД
```mysql
mysql> create database dz;
Query OK, 1 row affected (0.05 sec)

mysql> use dz;
Database changed

mysql> create table colors (id int(16) NOT NULL auto_increment, name varchar(256), value varchar(256), PRIMARY KEY (id));
Query OK, 0 rows affected, 1 warning (0.56 sec)

mysql> INSERT INTO colors (name, value) VALUES ('Arlekin', '#44944A'), ('Scarlet', '#FF2400'), ('Bistr', '#3D2B1F'), ('Bronze', '#CD7F32'), ('Vanilla', '#D5713F');
Query OK, 5 rows affected (0.06 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM colors
    -> ;
+----+---------+---------+
| id | name    | value   |
+----+---------+---------+
|  1 | Arlekin | #44944A |
|  2 | Scarlet | #FF2400 |
|  3 | Bistr   | #3D2B1F |
|  4 | Bronze  | #CD7F32 |
|  5 | Vanilla | #D5713F |
+----+---------+---------+
5 rows in set (0.00 sec)

mysql> exit
```
#### ШАГ 5. Создание скрипта php

```bash
$ docker exec -it nginx-php bash
bash-5.0# ls
index.php
bash-5.0# cat > index.php
<?php                         

//phpinfo();

$link = mysqli_connect('mysql', 'root', '123');
if (!$link) {
    die('Ошибка соединения: ' . mysqli_error());
}
echo 'Успешно соединились';
mysqli_close($link);
$
```
![Error](./images/error.png)
