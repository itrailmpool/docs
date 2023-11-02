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
```sh
[2023-11-02 19:42:24.2920] [I] [bitcoin01] Waiting for daemons to come online ...
[2023-11-02 19:42:34.2940] [I] [bitcoin01] Waiting for daemons to come online ...
[2023-11-02 19:44:26.2234] [I] [bitcoin01] All daemons online
[2023-11-02 19:47:22.8341] [I] [bitcoin01] All daemons synched with blockchain
```
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
2023-11-02T16:46:02Z UpdateTip: new best=00000000000000000000129a9255ca1863a9e4e85d4dceccd6b0ecb6eb259420 height=814957 version=0x3fffe000 log2_work=94.513868 tx=912558586 date='2023-11-02T10:05:04Z' progress=0.999924 cache=27.5MiB(196973txo)
2023-11-02T16:46:13Z UpdateTip: new best=000000000000000000007e184e9c0956c7a9b61bb4010a0e043df9c88dbb956f height=814958 version=0x20008000 log2_work=94.513882 tx=912562050 date='2023-11-02T10:07:18Z' progress=0.999924 cache=29.0MiB(206428txo)
2023-11-02T16:46:33Z UpdateTip: new best=0000000000000000000192878b71362fdcfe3f7a6c7d23063d9b4df0729325a1 height=814959 version=0x24cba000 log2_work=94.513895 tx=912565852 date='2023-11-02T10:37:58Z' progress=0.999930 cache=32.3MiB(219773txo)
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
##### 4. После завершения синхронизации ноды убедитесь, что пул работает:
Возможны кейсы, когда пул падает, не дождавшись ответа от ноды:
```sh
[2023-11-02 19:44:26.2234] [I] [bitcoin01] All daemons online
[2023-11-02 19:47:22.8341] [I] [bitcoin01] All daemons synched with blockchain
[2023-11-02 19:49:03.6535] [E] [bitcoin01] Init RPC failed: The request was canceled due to the configured HttpClient.Timeout of 100 seconds elapsing., The request was canceled due to the configured HttpClient.Timeout of 100 seconds elapsing., The request was canceled due to the configured HttpClient.Timeout of 100 seconds elapsing.
```
В таких случаях пут нужно перезапустить.
