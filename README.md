###  OTUS Linux Professional Lesson #9 | Subject: Размещаем свой RPM в своем репозитории
#### Домашнее задание

#### Цель:
Научиться создавать свой RPM;
Научится создавать свой репозиторий с RPM;


Описание/Пошаговая инструкция выполнения домашнего задания:
Для выполнения домашнего задания используйте методичку
Практика управления пакетами

#### Что нужно сделать?

- создать свой RPM (можно взять свое приложение, либо собрать к примеру апач с определенными опциями);
- создать свой репо и разместить там свой RPM;
- реализовать это все либо в вагранте, либо развернуть у себя через nginx и дать ссылку на репо.

Задание со звездочкой* реализовать дополнительно пакет через docker

В чат ДЗ отправьте ссылку на ваш git-репозиторий . Обычно мы проверяем ДЗ в течение 48 часов.

Если возникнут вопросы, обращайтесь к студентам, преподавателям и наставникам в канал группы в Telegram.

Удачи при выполнении!

Критерии оценки:
Статус "Принято" ставится, если сделаны репо и рпм.
Дополнительно можно сделать докер образ.

#### Выполнение

Для данного задания нам понадобятся следующие установленные пакеты:
````
sudo apt install -y dpkg-dev build-essential zlib1g-dev libpcre3 libpcre3-dev unzip
````

Директория для сборки
````
mkdir /home/vagrant/custom-nginx && cd /home/vagrant/custom-nginx
````
Качаем исходник:
````
apt source nginx
````
Добавляем модуль ngx_brotli
````
cd nginx-1.18.0/debian/modules
git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
cd ngx_brotli/deps/brotli
mkdir out && cd out
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_CXX_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_INSTALL_PREFIX=./installed ..
cmake --build . --config Release --target brotlienc
````

Прописываем дополнительный модуль в конфиге сборки пакета
````
nano /home/vagrant/custom-nginx/nginx-1.18.0/debian/rules
````
Добавляем в секцию common_configure_flags, не забываем поставить \
````
##################
--add-module=$(MODULESDIR)/ngx_brotli
##################
````


````
nano /home/vagrant/custom-nginx/nginx-1.18.0/debian/changelog
````
меняем версию
nginx (1.18.0-6ubuntu14.4-custom) jammy; urgency=medium

скачиваем зависимости
````
apt-get build-dep nginx
````
Собираем пакет
````
cd /home/vagrant/custom-nginx/nginx-1.18.0/
dpkg-buildpackage -b
````
Убедимся, что пакеты создались:
````
ll /home/vagrant/custom-nginx
````

Теперь можно установить наш пакет и убедиться, что nginx работает:
````
 dpkg -i *.deb
 systemctl start nginx
 systemctl status nginx
````
Фиксируем обновления
````
apt-mark hold ngnix
````

Далее мы будем использовать его для доступа к своему репозиторию


#### ЗАДАНИЕ 2. Создать свой репозиторий и разместить там ранее собранный RPM

 Создаем каталог репозитория в дефолтном каталоге статики nginx:
```
 mkdir /var/www/html/repo
```
Копируем в него наши пакеты:
```
cp *.deb /var/www/html/repo
```
 Настраиваем nginx для листинга каталога резитория. Добавляем в файл `nano /etc/nginx/sites-enabled/default` в секцию `location /` директиву `autoindex on`

Проверяем синтаксис и перезапускаем nginx:
```
# nginx -t
# systemctl restart nginx
```
Проверяем что репозиторий доступен:
```
curl -a http://localhost/repo/
```

Качаем тестовый пакет и сканируем репозитарий на наличие пакетов 
````
wget https://downloads.percona.com/downloads/pmm2/2.41.2/binary/debian/jammy/x86_64/pmm2-client_2.41.2-6.jammy_amd64.deb
dpkg-scanpackages -m . > Packages
dpkg-scanpackages -m . | gzip > Packages.gz
````
Добавляем репозиторий в систему:

````
vi /etc/apt/sources.list
````
````
##################
deb [trusted=yes] http://localhost/repo/ /
##################
````
Тестируем установку с нашего репозитория. 
```
apt install pmm2-client
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  pmm2-client
0 upgraded, 1 newly installed, 0 to remove and 2 not upgraded.
Need to get 85.9 MB of archives.
After this operation, 197 MB of additional disk space will be used.
Get:1 http://localhost/repo  pmm2-client 2.41.2-6.jammy [85.9 MB]
Fetched 85.9 MB in 0s (330 MB/s)
Selecting previously unselected package pmm2-client.
(Reading database ... 125138 files and directories currently installed.)
Preparing to unpack .../pmm2-client_2.41.2-6.jammy_amd64.deb ...
Adding system user `pmm-agent' (UID 114) ...
Adding new group `pmm-agent' (GID 121) ...
Adding new user `pmm-agent' (UID 114) with group `pmm-agent' ...
Creating home directory `/usr/local/percona' ...
Unpacking pmm2-client (2.41.2-6.jammy) ...
Setting up pmm2-client (2.41.2-6.jammy) ...
Created symlink /etc/systemd/system/multi-user.target.wants/pmm-agent.service → /lib/systemd/system/pmm-agent.service.
Scanning processes...
Scanning candidates...
Scanning linux images...

Restarting services...
Service restarts being deferred:
Service restarts being deferred:
 /etc/needrestart/restart.d/dbus.service
 systemctl restart getty@tty1.service
 systemctl restart networkd-dispatcher.service
 systemctl restart systemd-logind.service
 systemctl restart user@1000.service

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```
Пакет установлен с нашего репозитория
```
 dpkg-query -l | grep pmm2-client
ii  pmm2-client                            2.41.2-6.jammy                          amd64        Percona Monitoring and Management Client
```


> [!NOTE]
> В случае, если потребуется обновить репозиторий (а это делается при каждом добавлении файлов) снова, нужно выполнить команду `dpkg-scanpackages -m . > Packages`









