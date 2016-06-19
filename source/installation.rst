.. _installation:

*******************
Инсталляция системы
*******************

Где взять
=========

Для всех
  Скачать необходимые инсталляционные пакеты можно в официальном техническом сообществе Microimpuls
  по ссылке http://forum.micro.im/ в разделе “Дистрибутивы и обновления ПО”.

Для инженеров Microimpuls
  При установке ПО на сервер через систему оркестровки все необходимые установочные пакеты актуальных версий
  скачиваются из репозитория автоматически.

Debian
======

Все модули smarty поставляются в виде установочных deb-пакетов и устанавливаются утилитой dpkg.
Обязательным модулем является ``smarty-base``.

Для работы необходим web-сервер ``nginx``, ``uwsgi`` и ``python 2.7``, а также несколько других пакетов.

Установка требуемых пакетов
---------------------------

Установка через ``apt-get``:
::
    sudo apt-get install nginx uwsgi python2.7 python-dev libmysqlclient-dev
    libtiff4-dev libjpeg8-dev zlib1g-dev libfreetype6-dev liblcms2-dev
    libwebp-dev tcl8.5-dev tk8.5-dev python-tk

Установка smarty и модулей
--------------------------

Установка через ``dpkg``:
::
    dpkg -i smarty*.deb

После установки пакетов smarty необходимо установить python-библиотеки через ``pip``:
::
    sudo pip install -r /usr/share/nginx/html/microimpuls/smarty/requirements.txt

После установки пакета smarty-base создается конфигурационный файл для nginx в ``/etc/nginx/sites-available/smarty``.
По-умолчанию настроен на домен ``smarty.example.com`` и слушает порт ``80`` и ``8080``.

Файлы Web-приложения smarty размещаются в ``/usr/share/nginx/html/microimpuls/smarty``.

Установка схемы базы данных
---------------------------

Первичная установка схемы базы данных осуществляется командой:
::
    python /usr/share/nginx/html/microimpuls/smarty/manage.py migrate --settings=settings.<settings filename>

*<settings filename>* - имя файла настроек smarty, в котором должны быть установлены параметры подключения к БД
(см. :ref:`Системные настройки <system-settings>`).

Обновление версии
-----------------

*Перед обновлением версии, пожалуйста, сделайте резервную копию конфигурации, файлов smarty, а также дамп базы данных.*

Обновление устанавливается командой ``dpkg``:
::
    dpkg -i smarty*.deb

Доустановка новых или обновление требуемых python-библиотеки через ``pip``:
::
    sudo pip install -r /usr/share/nginx/html/microimpuls/smarty/requirements.txt

Миграция схемы БД осуществляется командой:
::
    python /usr/share/nginx/html/microimpuls/smarty/manage.py migrate --settings=settings.<settings filename>

Отличия release и dev версий
----------------------------

Пакеты smarty с суффиксом ``-dev`` позволяют установить на release-сервер дополнительную версию smarty в отдельные
директории с отдельным конфигурационным файлом, который может быть настроен на отдельную БД.
Это позволяет на одном сервере тестировать новую версию smarty без модификации и остановки production-версии.

Запуск, перезапуск и остановка
------------------------------

Панель администрирования доступна по адресу http://smarty.example.com, этот же адрес следует использовать
для подключения приложений для устройств как адрес API.

Общение Web-сервера с приложением осуществляется по интерфейсу WSGI (uWSGI).
Для запуска используется init.d скрипт: ``/etc/init.d/uwsgi``, доступные аргументы:
::
    # /etc/init.d/uwsgi
    Usage: /etc/init.d/uwsgi {start|stop|restart}

Файлы логов по-умолчанию сохраняются в ``/var/log/nginx/microimpuls/smarty/``, включена ротация логов через logrotate.d.

CentOS
======

Скрипт установки для fabric и примеры конфигурации на CentOS: https://github.com/microimpuls/smarty-centos
