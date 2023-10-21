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