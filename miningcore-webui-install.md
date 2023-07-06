# Установка Miningcore Web UI
### Шаг 1. Клонируем репозиторий
```sh
cd /home/user/
git clone https://github.com/itrailmpool/miningcore.web.ui.git
```
### Шаг 2. Меняем url пула
```sh
nano webui/js/miningcore.js
```
В открывшемся файле преобразуем 33 строку следующим образом:
```sh
var API            = "http://192.168.XX.XX:4000/" + "api/";
```
`192.168.XX.XX` - адрес вашей машины в локальной сети
