# Issues resolving FAQ
Сюда можно добавлять список проблем которые возникают/могут возникнуть и пути их решения

### Bitcoin core нода упала с ошибкой "libevent: Error from accept() call: Too many open files"
Ошибка "Too many open files" указывает на то, что приложение достигло максимального количества файлов и сетевых соединений, которые ему разрешено открывать.
Ошибку можно увидеть в логах тут: '/home/user/log/bitcoin/debug.log'
```sh
tail /home/user/log/bitcoin/debug.log
```
#### Как лечить:
##### 1. Сначала останавливаем и перезапускаем пул.
Получает PID пула:
```sh
netstat -tulpn | grep 4100
```
Убиваем процесс (`PID` заменить на PID пула полученный ранее):
```sh
kill -9 PID
```
Перезапускаем пул:
```sh
cd /home/user/miningcore/build/
nohup ./Miningcore -c config.json &
```
Пул должен запуститься, но будет ожидать когда станет доступна нода 
##### 2. Запускаем Bitcoin core ноду.
Запустим  ноду:
```sh
bitcoind -conf=/home/user/.bitcoin/bitcoin.conf
```
Убедимся, что нода запущена:
```sh
tail -f /home/user/log/bitcoin/debug.log
```
В логе должен быть виден процесс подключения к peer'ам сети биткоин и процесс синхронизации
```sh
2023-10-29T11:41:35Z New outbound peer connected: version: 70016, blocks=814368, peer=2319 (block-relay-only)
2023-10-29T11:41:37Z CreateNewBlock(): block weight: 3992630 txs: 3078 fees: 11527902 sigops 9051
2023-10-29T11:41:43Z CreateNewBlock(): block weight: 3992804 txs: 3111 fees: 11578460 sigops 9105
2023-10-29T11:41:52Z UpdateTip: new best=00000000000000000004019a579ac8c5cc7df8cb1c1851108d16e5c895d61171 height=814369 version=0x25756000 log2_work=94.505828 tx=910871302 date='2023-10-29T11:41:07Z' progress=1.000000 cache=96.4MiB(596234txo)
```
Как только процесс синхронизации завершится (`progress=1.000000`) пул начнет раздавать задания майнерам.
##### 3. Увеличить текущий лимит открытых файлов
Проверить текущий лимит открытых файлов можно так:
```sh
ulimit -n
```
увеличить лимит можно так:
```sh
ulimit -n 2048
```
Это увеличит лимит до следующего перезапуска системы или пока вы не вернете его обратно.

Чтобы увеличить лимит навсегда откройте файл /etc/security/limits.conf в текстовом редакторе с правами администратора.
Добавьте следующие строки в конец файла, заменив username на имя пользователя, под которым запущен процесс Bitcoin Core:
```sh
username soft nofile 4096
username hard nofile 10240
```
Сохраните файл и перезагрузите систему, чтобы применить изменения.
