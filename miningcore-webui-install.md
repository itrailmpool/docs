# Установка Miningcore Web UI
### Шаг 1. Клонируем репозиторий
```sh
cd /home/user/
git clone https://github.com/itrailmpool/miningcore.web.ui.git
```
### Шаг 2. Меняем url пула
```sh
nano miningcore.web.ui/js/miningcore.js
```
В открывшемся файле преобразуем 9 строку следующим образом:
```sh
var API            = "http://192.168.XX.XX:4000/" + "api/";
```
`192.168.XX.XX` - адрес вашей машины в локальной сети

### Шаг 3. Настраиваем nginx
```sh
sudo nano /etc/nginx/conf.d/miningcoreweb.conf
```
```sh
server {
	listen 0.0.0.0:4001;

	location / {
		root /home/user/miningcore.web.ui;
		index index.html;
	}
}
```
Cохраняем и перезапускаем nginx
```sh
sudo systemctl restart nginx.service
```
