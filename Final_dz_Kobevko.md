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
Теперь Docker установлен, демон запущен, и процесс будет запускаться при загрузке системы.  Убедимся, что процесс запущен:
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
Произвожу те же действия для MySQL
```bash
$ docker search mysql
$ docker pull mysql
$ docker run -d --name mysql -p 3306:3306 mysql
```