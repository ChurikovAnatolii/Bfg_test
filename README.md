# Bfg_test

<br>

***Задание выполнено с использованием ansible и vagrant. В качестве ОС на виртуальной машине используется образ ubuntu 18.04. В связи с тем, что не удалось запустить предложенное приложение( при выполнении описанных в readme действий приложение не запускалось с кучей ошибок, не говоря уже о его запуске через wsgi совместно с gunicorn) , было принято решение по реализации всей инфраструкты вокруг приложения ( установка всех зависимостей, создание изолированного окружения, создание базы данных, рестора таблиц, создания супервизора), а для демонстрации работы со связкой nginx+gunicorn+aiohttp в этом же плейбуке было реализовано разворачивание из git тестового приложения - переменная, которая включает или отключает этот функционал находится в defaults/main.yml*** 

---

<br>

## Описание запуска стенда
<br>

1. Для запуск стенда необходимы следующие компоненты, установленные на хостовой машине: 
 - Virtualbox
 - Ansible
 - Vagrant

 2. Пулим репо, заходим в папку с проектом и запускаем стенд командой start.sh. Ansible playbook для конфигурайии виртуальной машины запустится в автомате. По умолчанию для гостевой машины установлены следующие параметры по использованию ресурсов хоста:
 - Количество цп: 2
 - ОЗУ: 2048 мб  

<br>

## Описание файлов стенда.
<br>

 1. В файле defaults/main.yml описаны основные переменные, влияющие на работу стенда:
 - test_app: true - запускать ли тестовое приложение с unicorn для демонстрации работы связки nginx + gunicorn + python app;
- python_from_source: true - если необходимо компилировать питон из исходников, то необходимо ставить в true;
- python_source_url: https://www.python.org/ftp/python/3.6.13/Python-3.6.13.tgz - ссылка на архив с исходником;
- python_version_name: Python-3.6.13 - имя папки в которой будем подготавливать
исходник. Имя пакета без расширения;
- python_dir: /home/bfg/.python - папка куда установится питон;

- db_user: root - имя пользователя БД;
- db_password: 12345 - пароль;
- db_name: stack_exchange - имя базы данных;

2. В папке /templates лежат шаблоны следующих конфигов:

- gunicorn_stack.service.j2 - systemd unit для запуска gunicorn для stack_over_search как сервиса;
- gunicorn_tst.service.j2 - юнит для тестового приложения; 
- nginx.conf.j2 - конфиг nginx;
- root_cnf.j2 - конфиг для подключения к базе по ssh;
- settings.ini.j2 - конфиг stack_oversearch

3. В корне находятся следующие файлы:

- Hosts.yml - inventory для плейбука;
- ansible.cfg - файл настройки ansible;
- playbook.yml - основной плейбук


<br>

## Описание работы стенда.
<br>

1. После запуска создаем пользователя и группу bfg, забираем приложение из гита, устанавливаем зависимости.
2. Запускаем MySQL и Redis как сервисы, копируем конфиг ngninx из шаблона.
3. Далле запускаем таски по сбору питона из исходников( папки и версию указываются через переменные)- будут собраны только в том случае, если переменная python_from_source: true.
4. Далее конфигурим базу данных: 
- Апдейтим пароль и привилегии для пользователя root;
- Создаем базу данных;
- Ресторим таблицы из tables.sql;
- Сохраняем конфиг для подключения к БД по ssh;
5. Создаем кружение для приложения stack_over_search:
- создаем папки для конфигов и логов;
- закидываем конфиг из шаблона;
- создаем виртуальное окружение с помощью скомпилированного python.
- копиуем конфиг systemd unit для gunicorn из шаблона.
- запускаем systemd gunicorn для приложения с 5 воркерами c помощью handlers.
6. Все операции, указанные выше повторяем для тестового приложения.
---
**Обратите внимание!!! В связи с компиляцией питона из исходников - конфигурация может занять сущственное время- наберитесь терпения!!!**
