# Руководство по установке и запуску блокчейн нод

Предполагается, что у вас уже установлен Oracle Linux 8 и у вас есть права root или возможность использовать команду sudo.

## Bitcoin нода
### Шаг 1. Обновите свою систему
```sh
sudo dnf update -y
```
### Шаг 2. Установите необходимые зависимости
```sh
sudo dnf install -y wget gcc-c++ libtool make autoconf openssl-devel libevent-devel boost-devel libdb-devel automake python3 zeromq-devel
```

#### Berkeley DB
Oracle Linux использует "Berkeley DB" для предоставления функциональности базы данных, которая необходима для Bitcoin Core. Эта библиотека представлена в пакетах libdb и libdb-devel. Но Bitcoin Core требуется версия этой библиотеки 4.8, в то время как в репозитории Oracle Linux может быть более новая версия, что может вызвать проблемы.

Нужно вручную скомпилировать и установить версию Berkeley DB 4.8:
##### 1. Загрузите исходный код Berkeley DB 4.8:
```sh
wget http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz
```
##### 2. Распакуйте архив и перейдите в директорию с исходным кодом:
```sh
tar -xvf db-4.8.30.NC.tar.gz
```
##### 3. Исправление конфликта имен в Berkeley DB для успешной компиляции:
```sh
sed s/__atomic_compare_exchange/__atomic_compare_exchange_db/g -i db-4.8.30.NC/dbinc/atomic.h
```
##### 4. Создайте каталог для установки:
```sh
mkdir -p /opt/db-4.8.30.NC/
```
##### 5. Сконфигурируйте и установите Berkeley DB 4.8
Команда ../dist/configure выполняет скрипт, который подготавливает исходный код для сборки на вашей системе.
```sh
cd db-4.8.30.NC/build_unix/
../dist/configure --enable-cxx --disable-shared --with-pic --prefix=/opt/db-4.8.30.NC
```

- `--enable-cxx` этот флаг включает поддержку C++.
- `--disable-shared` этот флаг отключает сборку shared libraries. Это значит, что все программы, которые будут собраны, будут статически связаны с библиотеками, которые они используют.
- `--with-pic` этот флаг включает создание "position-independent code" (PIC). PIC используется для создания shared libraries, которые могут быть загружены в произвольное место в адресном пространстве.
- `--prefix=/opt/db-4.8.30.NC` этот флаг указывает каталог, в который будут установлены все файлы после сборки. В данном случае, все файлы будут установлены в каталог /opt/db-4.8.30.NC.

Компиляция и сборка проекта:
```sh
make -j4
```
> Note: Опция -j сообщает make, сколько заданий (jobs) можно выполнять одновременно. В нашем случае есть процессор с 4 ядрами, make -j4 позволит использовать все ядра, чтобы ускорить процесс сборки.
```sh
sudo make install
```
##### 6. Очистка
Удалите загруженные файлы, они больше не понадобятся:
```sh
cd ~
rm -rf db-4.8.30.NC*
```
Дополнительно можно удалить загруженную документацию Berkeley DB (78 Mb):
```sh
rm -rf /opt/db-4.8.30.NC/docs
```

### Шаг 3. Загрузите исходный код Bitcoin Core и распакуйте архив
```sh
wget https://bitcoincore.org/bin/bitcoin-core-23.0/SHA256SUMS && \
    wget https://bitcoincore.org/bin/bitcoin-core-23.0/SHA256SUMS.asc && \
    wget https://bitcoincore.org/bin/bitcoin-core-23.0/bitcoin-23.0.tar.gz && \
    grep " bitcoin-23.0.tar.gz\$" SHA256SUMS | sha256sum -c - && \
    tar -xzf *.tar.gz
```
(Проверьте и укажите актуальную на момент установки версию https://bitcoincore.org/en/download/)
### Шаг 4. Модификация конфигурационных файлов для корректной сборки Bitcoin Core
```sh
cd bitcoin-23.0
sed -i '/AC_PREREQ/a\AR_FLAGS=cr' src/univalue/configure.ac
sed -i '/AX_PROG_CC_FOR_BUILD/a\AR_FLAGS=cr' src/secp256k1/configure.ac
sed -i s:sys/fcntl.h:fcntl.h: src/compat.h
```
### Шаг 5. Компиляция и установка Bitcoin Core
Команда `./autogen.sh` подготавливает проект к сборке, генерируя необходимые файлы и скрипты:
```sh
./autogen.sh
```
Команда `./configure` запускает скрипт конфигурации, который готовит среду для сборки из исходного кода:
```sh
./configure LDFLAGS=-L`ls -d /opt/db*`/lib/ CPPFLAGS=-I`ls -d /opt/db*`/include/ \
    --prefix=/opt/bitcoin-23 \
    --mandir=/usr/share/man \
    --disable-tests \
    --disable-bench \
    --disable-ccache \
    --with-gui=no \
    --with-utils \
    --with-libs \
    --with-daemon
```

- `LDFLAGS=-Lls -d /opt/db*/lib/ и CPPFLAGS=-Ils -d /opt/db*/include/` эти параметры задают пути для библиотек и заголовочных файлов Berkeley DB.
- `--prefix=/opt/bitcoin-23` указывает, куда будут установлены файлы после сборки.
- `--mandir=/usr/share/man` Этот параметр указывает, где будут установлены файлы руководства (man pages).
- `--disable-tests` отключает сборку модулей тестирования.
- `--disable-bench` отключает сборку бенчмарков.
- `--disable-ccache` отключает использование ccache, который может ускорить повторные сборки за счет кеширования результатов предыдущих компиляций.
- `--with-gui=no` отключает сборку графического интерфейса пользователя (GUI).
- `--with-utils` включает сборку утилит Bitcoin.
- `--with-libs` включает сборку библиотек Bitcoin.
- `--with-daemon` включает сборку биткоин-демона (bitcoind).

После выполнения этой команды, вы можете запустить make для сборки Bitcoin Core.
```sh
make -j4
```
```sh
sudo make install
```
Процесс установки может занять некоторое время, в зависимости от производительности системы. В конце процесса bitcoin-cli и bitcoind будут установлены.

Это можно проверить так:
```sh
bitcoind --version
```
```sh
bitcoin-cli --version
```
Команды должны вернуть:
```sh
Bitcoin Core RPC client version v23.0.0
Copyright (C) 2009-2022 The Bitcoin Core developers

Please contribute if you find Bitcoin Core useful. Visit
<https://bitcoincore.org/> for further information about the software.
The source code is available from <https://github.com/bitcoin/bitcoin>.

This is experimental software.
Distributed under the MIT software license, see the accompanying file COPYING
or <https://opensource.org/licenses/MIT>
```
Либо возпользоваться командой whereis и посмотреть, возвращает ли она путь к bitcoind и bitcoin-cli.
```sh
whereis bitcoind
whereis bitcoin-cli
```
При установке в кастомный каталог может потребоваться явно указать путь к bitcoin-cli и bitcoind:
```sh
export PATH=$PATH:/home/user/bitcoin/bin
```
Чтобы изменения были постоянными и bitcoin-cli и bitcoind были доступен в $PATH при каждом запуске терминала, вы можете добавить эту команду в файл .bashrc или .bash_profile в вашем домашнем каталоге:
```sh
echo 'export PATH=$PATH:/home/user/bitcoin/bin' >> ~/.bashrc
```
### Шаг 6. Создание конфигурационного файла bitcoin.conf
Файл bitcoin.conf — это конфигурационный файл для bitcoind. Он содержит различные настройки для выполнения ноды Bitcoin. Основная структура файла представляет собой простой текстовый файл, в котором каждая строка представляет собой ключевую пару ключ=значение.

Перейдите в каталог /home/user/bitcoin-23 и создайте файл bitcoin.conf:
```sh
cd /home/user/bitcoin-23
nano bitcoin.conf
```
Добавьте следующие парамтеры и сохраните файл:
```sh
daemon=1
prune=3000
rpcuser=bitcoin
rpcpassword=bitcoin
zmqpubhashblock=tcp://127.0.0.1:15101
```
`TODO: update and add description for all params `

### Шаг 7. Запуск Bitcoin Core
```sh
bitcoind -daemon -conf=/home/user/bitcoin-23/bitcoin.conf
```
Bitcoin Core будет работать в фоновом режиме. Вы можете взаимодействовать с ней, используя bitcoin-cli.
`TODO: add  bitcoin-cli examples `
