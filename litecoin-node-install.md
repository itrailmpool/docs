# Установка Litecoin ноды
### Шаг 1. Обновите свою систему
```sh
sudo dnf update -y
```
### Шаг 2. Установите необходимые зависимости
```sh
sudo dnf install -y gcc-c++ libtool make autoconf automake libevent-devel boost-devel libdb4-devel libdb4-cxx-devel python3 fmt zeromq-devel
```

#### Berkeley DB

Oracle Linux использует Berkeley DB для предоставления функциональности базы данных, которая необходима для Litecoin. Litecoin требуется версия этой библиотеки 4.8, в то время как в репозитории Oracle Linux может быть более новая версия, что может вызвать проблемы.

Нужно вручную скомпилировать и установить версию Berkeley DB 4.8:

- [Установка Berkeley DB](./berkeley-db-install.md)

### Шаг 3. Скачивание архива с Litecoin и проверка его цифровой подписи
```sh
wget https://download.litecoin.org/litecoin-0.21.2/linux/litecoin-0.21.2-x86_64-linux-gnu.tar.gz
wget https://download.litecoin.org/litecoin-$0.21.2/linux/litecoin-0.21.2-x86_64-linux-gnu.tar.gz.asc
gpg --keyserver pgp.mit.edu --recv-key 0x3620E9D387E55666
gpg --verify litecoin-0.21.2-x86_64-linux-gnu.tar.gz.asc
```
(Проверьте и укажите актуальную на момент установки версию https://blog.litecoin.org/)

### Шаг 4. Распаковка архива и установка
```sh
tar xfz litecoin-0.21.2-x86_64-linux-gnu.tar.gz
sudo mv litecoin-0.21.2/bin/* /opt/litecoin-0.21.2/bin/
```

Потребуется явно указать путь к litecoin-cli и litecoind:
```sh
export PATH=$PATH:/opt/litecoin-0.21.2/bin
```
Чтобы изменения были постоянными и litecoin-cli и litecoind были доступен в $PATH при каждом запуске терминала, добавьте эту команду в файл .bashrc или .bash_profile в вашем домашнем каталоге:
```sh
echo 'export PATH=$PATH:/opt/litecoin-0.21.2/bin' >> ~/.bashrc
```

После этого шага litecoin-cli и litecoind будут установлены и готовы к использованию. Можно проверить это так:
```sh
litecoind --version
```
```sh
litecoin-cli --version
```
Команды должны вернуть:
```sh
TODO: add example
```

### Шаг 5. Очистка
Удалите загруженные файлы, они больше не понадобятся:
```sh
cd ~
rm -rf litecoin-* /root/.gnupg/
```

### Шаг 6. Создание конфигурационного файла litecoin.conf
litecoin.conf — это конфигурационный файл для litecoind. Он содержит различные настройки для выполнения ноды Litecoin. Основная структура файла представляет собой простой текстовый файл, в котором каждая строка представляет собой ключевую пару ключ=значение.

Создайте и перейдите в рабочий каталог ноды:
```sh
mkdir -p /home/user/.litecoin
cd /home/user/.litecoin
```
Создайте конфигурационный файл litecoin.conf:
```sh
nano litecoin.conf
```

Добавьте следующие парамтеры и сохраните файл:
```sh
# Enable pruning to save disk space
prune=1000

# Run in the background as a daemon and accept commands
daemon=1

# Set the directory for the blocks
datadir=/home/user/.litecoin

# Set the directory for the logs
debuglogfile=/home/user/log/litecoin/debug.log

# RPC Settings
rpcuser=litecoin
rpcpassword=litecoin
rpcbind=0.0.0.0
rpcallowip=192.168.86.0/24
rpcport=9332

# Server mode
server=1

# ZeroMQ notification options
zmqpubhashblock=tcp://0.0.0.0:29332
```
Описание параметров конфигурационного файла:
- `prune=1000` параметр позволяет вам обрезать (или "prune") блокчейн, сохраняя только определенное количество МБ данных. В данном случае, Litecoin будет хранить только последние 1000 МБ данных блокчейна, удаляя старые блоки, чтобы сэкономить место на диске.
- `daemon=1` Запускает Litecoin в режиме демона, то есть в фоновом режиме. Litecoin будет продолжать работать, даже если вы закроете консоль, с которой его запустили.
- `datadir=/path/to/your/data/directory` устанавливает каталог, где Litecoin будет хранить данные блокчейна. Замените /path/to/your/data/directory на реальный путь к каталогу.
- `debuglogfile=/path/to/your/log/directory/debug.log` устанавливает каталог, где Litecoin будет сохранять логи отладки. Замените /path/to/your/log/directory/debug.log на реальный путь к каталогу и имени файла.
- `rpcuser=litecoin` и `rpcpassword=litecoin` устанавливает имя пользователя и пароль для JSON-RPC подключений. В этом случае, имя пользователя и пароль оба установлены как "litecoin".
- `rpcbind=0.0.0.0` указывает, на каком IP-адресе Litecoin будет слушать входящие RPC-подключения. Значение 0.0.0.0 означает, что он будет слушать на всех доступных IP-адресах.
- `rpcallowip=192.168.86.0/24` разрешает RPC подключения только с IP-адресов в указанном диапазоне.
- `rpcport=9332` устанавливает порт для RPC подключений. В данном случае, порт установлен на 9332.
- `server=1` запускает Litecoin в режиме сервера, что позволяет ему принимать JSON-RPC команды.
- `zmqpubhashblock=tcp://0.0.0.0:29332` включает оповещения ZeroMQ для новых блоков. Это нужно, чтобы внешнее приложение (майнинг-пул), могло получать уведомления каждый раз, когда нода Litecoin обнаруживает новый блок.

### Шаг 7. Запуск Litecoin
Перед запуском создайте каталог для логов указанные в конфигурационном файле:
```sh
mkdir -p /home/user/log/litecoin/
```
Запускаем ноду Litecoin указывая путь к конфигурационному файлу:
```sh
litecoind -conf=/home/user/.litecoin/litecoin.conf
```
Litecoin будет работать в фоновом режиме. Вы можете взаимодействовать с ней, используя litecoin-cli интерфейс командной строки (CLI).

getblockchaininfo это команда RPC, которая возвращает различную статистическую информацию о текущем состоянии блокчейна. 
```sh
litecoin-cli -rpcuser=litecoin -rpcpassword=litecoin getblockchaininfo
```
Пример ответа ноды:
```sh
{
TODO add example
}
```

## Полезные ссылки
- [Github репозиторий litecoin ](https://github.com/litecoin-project/litecoin/blob/master/doc/README.md)
- [Описание релиза 0.21.2](https://blog.litecoin.org/litecoin-core-v0-212-release-282f5405aa11)
