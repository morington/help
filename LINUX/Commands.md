## Узнать какой процесс использует порт:

```
sudo lsof -i :<port>
```

_**Пример**: допустим, мы хотим узнать, кто использует порт_ `8080`

```
❯ sudo lsof -i :8080
COMMAND  PID      USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
python  8362 morington    7u  IPv4 106282      0t0  TCP localhost:http-alt->localhost:54888 (CLOSE_WAIT)
python  8362 morington    9u  IPv4 104610      0t0  TCP localhost:http-alt (LISTEN)
```

## Найти определенный процесс в системе:

```
ps afx | grep <name>
```


*Команда ps используется для отображения информации о процессах, запущенных в системе. Она выводит на экран список процессов, в том числе их идентификаторы (PID), имена команд, состояния, приоритеты и другие данные.*

_**Аргумент `a`**_ *указывает на то, что должны быть выведены все процессы, включая фоновые.*

_**Аргумент `f`**_ *указывает на то, что выходные данные должны быть разделены по процессам.*

_**Аргумент `x`**_ *указывает на то, что должны быть выведены только процессы, запущенные пользователем.*

```
❯ ps afx | grep postgresql
    692 pts/0    S+     0:00  |           \_ grep --color=auto postgresql
    672 ?        Ss     0:00 /usr/lib/postgresql/14/bin/postgres -D /var/lib/postgresql/14/main -c config_file=/etc/postgresql/14/main/postgresql.conf
```

*Также отображает с какими параметрами запущен процесс.*

- `-D /var/lib/postgresql/14/main` *- указывает на каталог, в котором находится база данных PostgreSQL.*
- `-c config_file=/etc/postgresql/14/main/postgresql.conf` *- указывает на файл конфигурации PostgreSQL.*