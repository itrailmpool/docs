# Litecoin нода
### Шаг 1. Обновите свою систему
```sh
sudo dnf update -y
```
### Шаг 2. Установите необходимые зависимости
```sh
sudo dnf install gcc-c++ libtool make autoconf automake libevent-devel boost-devel libdb4-devel libdb4-cxx-devel python3 fmt zeromq-devel
```

#### Berkeley DB

Oracle Linux использует Berkeley DB для предоставления функциональности базы данных, которая необходима для Litecoin. Litecoin требуется версия этой библиотеки 4.8, в то время как в репозитории Oracle Linux может быть более новая версия, что может вызвать проблемы.

Нужно вручную скомпилировать и установить версию Berkeley DB 4.8:

- [Установка Berkeley DB](./berkeley-db-install.md)

### Шаг 3. Загрузите исходный код Litecoin и распакуйте архив
```sh

```
(Проверьте и укажите актуальную на момент установки версию )
