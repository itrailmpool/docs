# Установка itail-viewer
На данном этапе предполагается, что все необходимые блокчейн ноды уже развенуты и полностью синхронизированы с сетью блокчейна и сам пул также запущен.

### Шаг 1. Установите необходимые зависимости
```sh
sudo dnf install java-17-openjdk-devel -y
```
### Шаг 2. Установите необходимые зависимости
Добавьте переменную окружения JAVA_HOME в PATH. Чтобы изменения были постоянными и JAVA_HOME были доступен в $PATH при каждом запуске терминала, добавьте эту команду в файл .bashrc или .bash_profile в вашем домашнем каталоге:
```sh
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-17.0.8.0.7-2.0.1.el8.x86_64/' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
```
Выполните:
```sh
source ~/.bashrc
```
### Шаг 3. Клонировать репозиторий приложения:
```sh
cd /home/user/
git clone https://github.com/itrailmpool/itrailmpool-viewer.git
```
### Шаг 4. Собираем приложение
```sh
mvn clean install
```
### Шаг 5.Запускаем приложение
```sh
nohup java -Djava.net.preferIPv4Stack=true -Dspring.profiles.active=prod -Denv=prod -jar ./target/itrailmpool-viewer.jar &
```
