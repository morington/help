# Настройка и конфигурация NATS

## System: Linux

Для начала необходимо проверить, чтобы в системе были установлены следующие пакеты:

* wget
* tar

## Загрузка NATS:

Для начала с помощью ссылки загрузим архив в систему:

```
wget https://github.com/nats-io/nats-server/releases/download/v2.9.15/nats-server-v2.9.15-linux-amd64.tar.gz
```

Распакуем:

```
tar -zxf nats-server-*.tar.gz
```

Копируем бинарный файл в каталог `**/usr/bin**`:

```
sudo cp nats-server-*-linux-amd64/nats-server /usr/bin
```

Проверим, что nats-server может быть запущен:

```
nats-server -v
```

Мы должны получить что-то надобие: `nats-server: v2.9.15`

Исходники можно удалить:

```
rm -rf nats-server-*linux-amd64 nats-server-*.tar.gz
```

## Запуск

Для разового запуска достаточно ввести команду: `nats-server`

Мы увидем логи запущеного сервиса NATS:

```
[153420] 2023/04/04 16:20:40.059157 [INF] Starting nats-server  
[153420] 2023/04/04 16:20:40.059246 [INF]   Version:  2.9.15  
[153420] 2023/04/04 16:20:40.059254 [INF]   Git:      [b91fa85]  
[153420] 2023/04/04 16:20:40.059264 [INF]   Name:     NDAXZI62G33F3IR6UGBMFCL4XMPPI6VDRCYDU4NP5OFSKYIH4TDQ63WQ  
[153420] 2023/04/04 16:20:40.059268 [INF]   ID:       NDAXZI62G33F3IR6UGBMFCL4XMPPI6VDRCYDU4NP5OFSKYIH4TDQ63WQ  
[153420] 2023/04/04 16:20:40.060083 [INF] Listening for client connections on 0.0.0.0:4222  
[153420] 2023/04/04 16:20:40.060292 [INF] Server is ready
```

Прерываем работу сервиса клавишами `Ctrl+C`.

Теперь выполним запуск сервиса NATS в качестве сервиса `systemctl`.

Создадим конфигурационный файл для сервера nats:

```
sudo mkdir /etc/nats
sudo nano /etc/nats/nats-server.conf
```

Впишем следующую конфигурацию:

```
store_dir: "/var/lib/nats"
listen: "127.0.0.1:4222"
log_file: /var/log/nats/nats.log

jetstream: {}
monitor_port:8222
```

Обозначение:
- **store_dir** - путь к файлу хранилища данных.
- **listen** - адрес и порт, на котором будет слушать сервер.
- **log_file** - путь к файлу с журналом (логи).
- **jetstream** - включаем JetStream.
- **monitor_port** - порт для мониторинга.

Создадим служебную учетную запись NATS:

```
sudo useradd -r -c 'NATS service' nats
```

Создадим каталоги и назначим в качестве их владельца созданную учетную запись:

```
sudo mkdir /var/log/nats /var/lib/nats
sudo chown nats:nats /var/log/nats /var/lib/nats
```

Создадим systemd юнит-файл:

```
sudo nano /etc/systemd/system/nats-server.service
```

```
[Unit]  
Description=NATS messaging server  
After=syslog.target network.target  
  
[Service]  
Type=simple  
ExecStart=/usr/bin/nats-server -c /etc/nats/nats-server.conf  
User=nats  
Group=nats  
LimitNOFILE=65536  
ExecReload=/bin/kill -HUP $MAINPID  
Restart=on-failure  
  
[Install]  
WantedBy=multi-user.target
```

Для разрешения автозапуска при старте системы вводим:

```
sudo systemctl enable --now nats-server
```

Для запуска (также в ручном режиме) используем:

```
sudo systemctl start nats-server
```

Статус можно посмотреть командой:

```
sudo systemctl status nats-server
```

## Установка natscli

- ArchLinux:

    ```
    yay -S natscli
    ```

- Debian/Ubuntu:

    ```
    wget https://github.com/nats-io/natscli/releases/download/v0.0.35/nats-0.0.35-amd64.deb
    
    sudo dpkg -i nats-0.0.35-amd64.deb
    rm -rf nats-*.deb
    ```

## Заметки

`nats --help` - выведет команды NATS

`nats stream ls -a` - просмотр скрытого слоя для KV

`sudo tail /var/log/nats/nats.log` - логи NATS
