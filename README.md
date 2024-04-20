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
