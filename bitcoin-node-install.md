# Установка Bitcoin ноды
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
- [Установка Berkeley DB](./berkeley-db-install.md)

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
    --prefix=/opt/bitcoin-23.0 \
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
- `--prefix=/opt/bitcoin-23.0` указывает, куда будут установлены файлы после сборки.
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
> Note: Опция -j сообщает make, сколько заданий (jobs) можно выполнять одновременно. В нашем случае есть процессор с 4 ядрами, make -j4 позволит использовать все ядра, чтобы ускорить процесс сборки.
```sh
sudo make install
```
Процесс установки может занять некоторое время, в зависимости от производительности системы. В конце процесса bitcoin-cli и bitcoind будут установлены.

При установке в кастомный каталог потребуется явно указать путь к bitcoin-cli и bitcoind:
```sh
export PATH=$PATH:/opt/bitcoin-23.0/bin
```
Чтобы изменения были постоянными и bitcoin-cli и bitcoind были доступен в $PATH при каждом запуске терминала, добавьте эту команду в файл .bashrc или .bash_profile в вашем домашнем каталоге:
```sh
echo 'export PATH=$PATH:/opt/bitcoin-23.0/bin' >> ~/.bashrc
```

Проверить, что bitcoin-cli и bitcoind успешно установлены можно так:
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

### Шаг 6. Создание конфигурационного файла bitcoin.conf
bitcoin.conf — это конфигурационный файл для bitcoind. Он содержит различные настройки для выполнения ноды Bitcoin. Основная структура файла представляет собой простой текстовый файл, в котором каждая строка представляет собой ключевую пару ключ=значение.

Создайте и перейдите в рабочий каталог ноды:
```sh
mkdir -p /home/user/.bitcoin
cd /home/user/.bitcoin
```
Создайте конфигурационный файл bitcoin.conf:
```sh
nano bitcoin.conf
```
Добавьте следующие парамтеры и сохраните файл:
```sh
# Enable pruning to save disk space
prune=2000

# Run in the background as a daemon and accept commands
daemon=1

# Set the directory for the blocks
datadir=/home/user/.bitcoin

# Set the directory for the logs
debuglogfile=/home/user/log/bitcoin/debug.log

# RPC Settings
rpcuser=bitcoin
rpcpassword=bitcoin
#rpcallowip=192.168.100.0/24
rpcport=8332

# Server mode
server=1

# ZeroMQ notification options
zmqpubhashblock=tcp://127.0.0.1:28332
```
Описание параметров конфигурационного файла:
- `prune=2000` параметр позволяет вам обрезать (или "prune") блокчейн, сохраняя только определенное количество МБ данных. В данном случае, Bitcoin Core будет хранить только последние 2000 МБ данных блокчейна, удаляя старые блоки, чтобы сэкономить место на диске.
- `daemon=1` Запускает Bitcoin Core в режиме демона, то есть в фоновом режиме. Bitcoin Core будет продолжать работать, даже если вы закроете консоль, с которой его запустили.
- `datadir=/path/to/your/data/directory` устанавливает каталог, где Bitcoin Core будет хранить данные блокчейна. Замените /path/to/your/data/directory на реальный путь к каталогу.
- `debuglogfile=/path/to/your/log/directory/debug.log` устанавливает каталог, где Bitcoin Core будет сохранять логи отладки. Замените /path/to/your/log/directory/debug.log на реальный путь к каталогу и имени файла.
- `rpcuser=bitcoin` и `rpcpassword=bitcoin` устанавливает имя пользователя и пароль для JSON-RPC подключений. В этом случае, имя пользователя и пароль оба установлены как "bitcoin".
- `rpcallowip=192.168.100.0/24` разрешает RPC подключения только с IP-адресов в указанном диапазоне.
- `rpcport=8332` устанавливает порт для RPC подключений. В данном случае, порт установлен на 8332.
- `server=1` запускает Bitcoin Core в режиме сервера, что позволяет ему принимать JSON-RPC команды.
- `zmqpubhashblock=tcp://0.0.0.0:28332` включает оповещения ZeroMQ для новых блоков. Это нужно, чтобы внешнее приложение (майнинг-пул), могло получать уведомления каждый раз, когда нода Bitcoin Core обнаруживает новый блок.

### Шаг 7. Запуск Bitcoin Core
Перед запуском создайте каталог для логов указанные в конфигурационном файле:
```sh
mkdir -p /home/user/log/bitcoin/
```
Запускаем ноду Bitcoin Core указывая путь к конфигурационному файлу:
```sh
bitcoind -conf=/home/user/.bitcoin/bitcoin.conf
```
Bitcoin Core будет работать в фоновом режиме. Вы можете взаимодействовать с ней, используя bitcoin-cli интерфейс командной строки (CLI).

[getblockchaininfo](https://bitcoincore.org/en/doc/23.0.0/rpc/blockchain/getblockchaininfo/) это команда RPC, которая возвращает различную статистическую информацию о текущем состоянии блокчейна. 
```sh
bitcoin-cli -rpcuser=bitcoin -rpcpassword=bitcoin getblockchaininfo
```
Пример ответа ноды:
```sh
{
  "chain": "main",
  "blocks": 229666,
  "headers": 797322,
  "bestblockhash": "0000000000000041e3ae6b19b0719282ea2241b897064418b1dc636012cbc714",
  "difficulty": 6695826.282596251,
  "time": 1365087839,
  "mediantime": 1365084588,
  "verificationprogress": 0.01854313338595879,
  "initialblockdownload": true,
  "chainwork": "000000000000000000000000000000000000000000000033d0acb81ec417c098",
  "size_on_disk": 1849974513,
  "pruned": true,
  "pruneheight": 219367,
  "automatic_pruning": true,
  "prune_target_size": 2097152000,
  "warnings": ""
}
```
## Создание кошлека и адресов
Создайте новый wallet для адресов ноды:
```sh
bitcoin-cli -rpcuser=bitcoin -rpcpassword=bitcoin createwallet ""
```

Создайте новый address:

```sh
bitcoin-cli -rpcuser=bitcoin -rpcpassword=bitcoin getnewaddress "" legacy
```
> Note: Bitcoin Core нода по умолчанию создает адреса нового формата (SegWit). Чтобы нода вернула адрес в легаси формате нужно явно это указать добавив `"" legacy`.
## Полезные ссылки
- [Bitcoin Core Config Generator](https://jlopp.github.io/bitcoin-core-config-generator/)
- [Github репозиторий Bitcoin Core](https://github.com/bitcoin/bitcoin/blob/master/doc/README.md)
