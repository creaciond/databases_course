# Установка Docker, развёртывание контейнера с базой, установка коннекта между базой в контейнере и MySQL Workbench

## Источники

Как сделать докер-контейнер и развернуть в контейнере MySQL сервер:

* [Start a Remote MySQL Server with Docker quickly (Cun Yang, Medium)](https://medium.com/@backslash112/start-a-remote-mysql-server-with-docker-quickly-9fdff22d23fd
).

Коннект контейнера и Workbench: 
* [StackOverflow с общим процессом](https://stackoverflow.com/questions/33827342/how-to-connect-mysql-workbench-to-running-mysql-inside-docker)
* StackOverflow с обходом проблемы PRIMARY KEY на UPDATE запросе к таблице mysql.user, ссылку на который я потеряла

## Инструкция

1) Зарегистрироваться на Docker Hub

2) Скачать себе:
* Docker Desktop (вместе с ним заодно установится Docker CLI /Command Line Interface/, который позволит взаимодействовать с докером в терминале)
* MySQL Workbench

3) Клонировать к себе рабочую версию контейнера с MySQL
Терминал: `docker pull mysql/mysql-server:latest`
(займёт время)

4) Проверить корректность:
Терминал: `docker images`
Выведутся все репозитории, которые есть у вас, среди них должен быть и mysql/mysql-server

5) Запустить контейнер и указать, через какой порт наш компьютер будет дружить с сервером, который спрятан в этом контейнере.
По умолчанию — порт 3306 компа встречается с портом 3306 сервера
Имя этому запуску можно дать любое, я назвала его mysqlroj

Терминал: `docker run --name=mysqlroj -e MYSQL_ROOT_HOST=% -p 3306:3306 -d mysql/mysql-server`

Дословный разбор команды: «Докер, запусти контейнер, который будет тихо работать на заднем плане, по образу из mysql/mysql-server, назови его mysqlroj, и дай ему системную настройку-переменную MYSQL_ROOT_HOST, в которую сложи указание, как дружат порты: 3306 компа + 3306 сервера»

6) Посмотреть логи (по имени контейнера, которое мы дали в `--name`):
Терминал: `docker logs mysqlroj`
Если всё будет хорошо, то там будут только инфосообщения типа «сервер запущен»

7) Скопировать сгенерированный пароль доступа
Терминал: `docker logs mysqlproj 2>&1 | grep GENERATED`
Появится строка с паролем. Пароль копируем
У меня был такоей: usabopOdoH-ULXoj3zaMOn(ukUS3

8) Подключить клиент MySQL к контейнеру:
Терминал: `docker exec -it mysqlproj mysql -uroot -p`
Нас попросят ввести пароль, вводим пароль из (7)
NB: символы отображаться не будут, это нормально

9) Система скажет, что мы логинимся первый раз и надо поменять пароль. Делаем это:
Терминал: `ALTER USER 'root'@'localhost' IDENTIFIED BY 'тут вводите свой пароль';`

В ответ придёт такое:
_Query OK, 0 rows affected (0.02 sec)_

10) Обновляем доступ к базе при помощи такого SQL-кода:
Терминал: `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'пароль';`
Терминал: `ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'пароль';`

11) Выходим из mysql
Терминал: `exit`

12) Перезапускаем контейнер:
Терминал: `docker restart mysqlproj`

13) Проверка! Просим Докер показать работающие контейнеры, в списке должен быть наш:
Терминал: `docker ps`

14) Подключение в Workbench должно работать по таким параметрам:
Host: 0.0.0.0
Port: 3306
