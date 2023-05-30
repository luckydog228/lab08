# lab08

```bash
├── client
│   ├── client.py
│   └── Dockerfile
├── docker-compose.yml
└── server
    ├── Dockerfile
    └── server.py

```

# client

### client.py
```python
import requests
import json
import time


while True:
    response = requests.get("http://0.0.0.0:1234/")
    print(response.json())
    time.sleep(5)

```
### Dockerfile
```dockerfile
#задаем базовый образ. Используем образ python
FROM python:3.9.6

#далее импортируем файлы, которые будем запускать
# файл client.py, который в докер-контейнере мы разместим в папке /client/
ADD client.py /client/
RUN pip3 install requests
#и установим директорию /server в качестве рабочей для контпейнера

WORKDIR /client/
```

# server

### server.py
```python
from http.server import *
import base64
import time

class Handler(BaseHTTPRequestHandler):
   
    def do_GET(self):
        print("Got request")
        self.send_response(200)
        self.send_header("Content-type", "application/json")
        self.end_headers()
        self.wfile.write(b'{"data" : "It\'s working yay"}')
       

class Server(HTTPServer):
    def __init__(self, server_address:  tuple[str, int], RequestHandlerClass, bind_and_activate=...) -> None:
        super().__init__(server_address, RequestHandlerClass, bind_and_activate)



if __name__ == "__main__":

    print("\n\n")
    server = Server(('0.0.0.0', 1234), Handler)
    print("Starting server %")
    server.serve_forever()
    server.server_close()
```

### Dockerfile
```dockerfile
#задаем базовый образ. Используем образ python
FROM python:3.9.6

#далее импортируем файлы, которые будем запускать
# файл server.py, который в докер-контейнере мы разместим в папке /server/
ADD server.py /server/
RUN pip install requests
#и установим директорию /server в качестве рабочей для контпейнера

WORKDIR /server/
```





# docker-compose.yml
```yml
# Файл docker-compose должен начинаться с тега версии.
# Мы используем "3" так как это - самая свежая версия на момент написания этого кода.

version: "3"

# Следует учитывать, что docker-composes работает с сервисами.
# 1 сервис = 1 контейнер.
# Сервисом может быть клиент, сервер, сервер баз данных...
# Раздел, в котором будут описаны сервисы, начинается с 'services'.

services:

  # Как уже было сказано, мы собираемся создать клиентское и серверное приложения.
  # Это означает, что нам нужно два сервиса.
  # Первый сервис (контейнер): сервер.
  # Назвать его можно так, как нужно разработчику.
  # Понятное название сервиса помогает определить его роль.
  # Здесь мы, для именования соответствующего сервиса, используем ключевое слово 'server'.

  server:
 
    # Ключевое слово "build" позволяет задать
    # путь к файлу Dockerfile, который нужно использовать для создания образа,
    # который позволит запустить сервис.
    # Здесь 'server/' соответствует пути к папке сервера,
    # которая содержит соответствующий Dockerfile.

    build: server/

    # Команда, которую нужно запустить после создания образа.
    # Следующая команда означает запуск "python ./server.py".

    command: python ./server.py

    # Вспомните о том, что в качестве порта в 'server/server.py' указан порт 1234.
    # Если мы хотим обратиться к серверу с нашего компьютера (находясь за пределами контейнера),
    # мы должны организовать перенаправление этого порта на порт компьютера.
    # Сделать это нам поможет ключевое слово 'ports'.
    # При его использовании применяется следующая конструкция: [порт компьютера]:[порт контейнера]
    # В нашем случае нужно использовать порт компьютера 1234 и организовать его связь с портом
    # 1234 контейнера (так как именно на этот порт сервер 
    # ожидает поступления запросов).

    ports:
      - 1234:1234

  # Второй сервис (контейнер): клиент.
  # Этот сервис назван 'client'.

  client:
    # Здесь 'client/ соответствует пути к папке, которая содержит
    # файл Dockerfile для клиентской части системы.

    build: client/

    # Команда, которую нужно запустить после создания образа.
    # Следующая команда означает запуск "python ./client.py".
 
    command: python ./client.py

    # Ключевое слово 'network_mode' используется для описания типа сети.
    # Тут мы указываем то, что контейнер может обращаться к 'localhost' компьютера.

    network_mode: host

    # Ключевое слово 'depends_on' позволяет указывать, должен ли сервис,
    # прежде чем запуститься, ждать, когда будут готовы к работе другие сервисы.
    # Нам нужно, чтобы сервис 'client' дождался бы готовности к работе сервиса 'server'.
 
    depends_on:
      - server
```
