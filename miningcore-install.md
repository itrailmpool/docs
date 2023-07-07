# Установка miningcore пула
На данном этапе предполагается, что все необходимые пулу блокчейн ноды уже развенуты и полностью синхронизированы с сетью блокчейна.

### Шаг 1. Установите необходимые зависимости

Сначала нужно загрузить и установить пакет `packages-microsoft-prod.rpm`. Этот пакет добавляет репозитории Microsoft в системы, основанные на Red Hat Enterprise Linux (RHEL), включая Oracle Linux 8:
```sh
wget https://packages.microsoft.com/config/rhel/8/packages-microsoft-prod.rpm
sudo dnf install -y ./packages-microsoft-prod.rpm
```
После установки пакет можно удалить:
```sh
rm packages-microsoft-prod.rpm
```
Установите остальные зависимости:
```sh
sudo dnf install -y dotnet-sdk-6.0 git cmake make openssl-devel pkgconfig boost-devel libsodium-devel zeromq-devel gcc-c++
```
### Шаг 2. Клонируем репозиторий пула:
```sh
cd /home/user/
git clone https://github.com/itrailmpool/miningcore.git
```

### Шаг 3. Настройка postgres
> Note: Предполагается, что postgres предаврительно установлен
```sh
sudo -u postgres psql

CREATE ROLE miningcore WITH LOGIN ENCRYPTED PASSWORD 'miningcore';
CREATE DATABASE miningcore OWNER miningcore;
\q
```
Выполнить скрипт для создания структуры БД [createdb.sql](createdb.sql):
```sh
sudo -u postgres psql -d miningcore -f createdb.sql
```

### Шаг 4. Собираем пул
```sh
cd ~/miningcore/src/Miningcore
dotnet publish -c Release --framework net6.0 -o ../../build
```
Процесс сборки может занять некоторое время, в зависимости от производительности системы.

### Шаг 5. Настройка конфигурационного файла
Содзадим конфигурационный файл `config.json` в директории ~/miningcore/build:
```sh
nano config.json
```
Копируем json конфигурацию и сохраняем файл:
```json
{
  "logging": {
    "level": "info",
    "enableConsoleLog": true,
    "enableConsoleColors": true,
    "logFile": "core.log",
    "apiLogFile": "api.log",
    "logBaseDirectory": "/home/user/log/miningcore",
    "perPoolLogFile": false
  },
  "banning": {
    "manager": "Integrated",
    "banOnJunkReceive": false,
    "banOnInvalidShares": false
  },
  "notifications": {
    "enabled": false,
    "email": {
      "host": "smtp.example.com",
      "port": 587,
      "user": "user",
      "password": "password",
      "fromAddress": "info@yourpool.org",
      "fromName": "pool support"
    },
    "admin": {
      "enabled": false,
      "emailAddress": "user@example.com",
      "notifyBlockFound": true
    }
  },
  "persistence": {
    "postgres": {
      "host": "192.168.100.2",
      "port": 5432,
      "user": "miningcore",
      "password": "miningcore",
      "database": "miningcore"
    }
  },
  "paymentProcessing": {
    "enabled": true,
    "interval": 600,
    "shareRecoveryFile": "recovered-shares.txt"
  },
  "api": {
    "enabled": true,
    "listenAddress": "0.0.0.0",
    "port": 4000,
    "metricsIpWhitelist": [],
    "rateLimiting": {
      "disabled": false,
      "rules": [
        {
          "Endpoint": "*",
          "Period": "1s",
          "Limit": 20
        }
      ],
      "ipWhitelist": []
    }
  },
  "pools": [
    {
      "id": "bitcoin01",
      "enabled": true,
      "coin": "bitcoin",
      "address": "1DnPPFQPrfyNTiHPXhDFyqNnW9T62GEhB1",
      "rewardRecipients": [
        {
          "address": "145aD9pJo8jcyhQDMU7L8mfmmiZzSugEYy",
          "percentage": 0.5
        }
      ],
      "blockRefreshInterval": 400,
      "jobRebroadcastTimeout": 10,
      "clientConnectionTimeout": 600,
      "banning": {
        "enabled": false,
        "time": 600,
        "invalidPercent": 50,
        "checkThreshold": 50
      },
      "ports": {
        "3052": {
          "listenAddress": "0.0.0.0",
          "difficulty": 0.02,
          "tls": false,
          "tlsPfxFile": "/var/lib/certs/mycert.pfx",
          "varDiff": {
            "minDiff": 0.01,
            "maxDiff": null,
            "targetTime": 15,
            "retargetTime": 90,
            "variancePercent": 30,
            "maxDelta": 500
          }
        }
      },
      "daemons": [
        {
          "host": "0.0.0.0",
          "port": 8332,
          "user": "bitcoin",
          "password": "bitcoin",
          "zmqBlockNotifySocket": "tcp://0.0.0.0:28332"
        }
      ],
      "paymentProcessing": {
        "enabled": true,
        "minimumPayment": 0.02,
        "payoutScheme": "PPLNS",
        "payoutSchemeConfig": {
          "factor": 2.0
        }
      }
    },
    {
      "id": "litecoin01",
      "enabled": true,
      "coin": "litecoin",
      "address": "ltc1qqsdkv6e2gsa4rpjwvc6cg4xlresw2s484kdsfe",
      "rewardRecipients": [
        {
          "address": "ltc1qznykl2a6sxff7tt8se4gfdhel0n4rmqhfwlt8s",
          "percentage": 0.5
        }
      ],
      "blockRefreshInterval": 400,
      "jobRebroadcastTimeout": 10,
      "clientConnectionTimeout": 600,
      "banning": {
        "enabled": false,
        "time": 600,
        "invalidPercent": 50,
        "checkThreshold": 50
      },
      "ports": {
        "3053": {
          "listenAddress": "0.0.0.0",
          "difficulty": 0.02,
          "tls": false,
          "tlsPfxFile": "/var/lib/certs/mycert.pfx",
          "varDiff": {
            "minDiff": 0.01,
            "maxDiff": null,
            "targetTime": 15,
            "retargetTime": 90,
            "variancePercent": 30,
            "maxDelta": 500
          }
        }
      },
      "daemons": [
        {
          "host": "0.0.0.0",
          "port": 9332,
          "user": "litecoin",
          "password": "litecoin",
          "zmqBlockNotifySocket": "tcp://0.0.0.0:29332"
        }
      ],
      "paymentProcessing": {
        "enabled": true,
        "minimumPayment": 0.02,
        "payoutScheme": "PPLNS",
        "payoutSchemeConfig": {
          "factor": 2.0
        }
      }
    }
  ]
}
```

### Шаг 6. Сохраняем и запускаем пул
```sh
./Miningcore -c config.json
```

## Описание параметров конфигурационного файла
`TODO add config description`
