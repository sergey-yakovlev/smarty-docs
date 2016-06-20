.. _system_configuration:

**********************
Системная конфигурация
**********************

.. _system-settings:

Системные настройки
===================

.. _add-smarty-configuration:

Добавление конфигурации
-----------------------

.. _smarty-config:

smarty
~~~~~~

После первичной установки базовый файл конфигурации smarty находится по адресу ``/etc/smarty/base.py``
(симлинк на ``/usr/share/nginx/html/microimpuls/smarty/settings/base.py``).

Основной файл конфигурации, используемый для production-режима работы - ``/etc/smarty/prod.py``.
На этот файл (или на другой используемый конфиг) должен указывать симлинк в ``/usr/share/nginx/html/microimpuls/smarty/settings/<setings name>.py``.
Именно в нем следует производить настройку smarty.

Конфигурация производится путем присваивания значений переменым на Python.

.. _nginx-config:

nginx
~~~~~

Конфигурация для **nginx** находится в файле ``/etc/nginx/sites.available/smarty``.

.. _uwsgi-config:

uwsgi
~~~~~

Конфигурация для **uwsgi** находится в файле ``/etc/microimpuls/smarty/uwsgi/smarty.uwsgi``,
на него (или на другой используемый конфиг) должен указывать симлинк в ``/usr/share/nginx/html/microimpuls/smarty/<uwsgi settings name>.uwsgi``.

.. _smarty-multiinstance:

Идеология инстансов
~~~~~~~~~~~~~~~~~~~

Для удобства конфигурации и размещения на одном сервере нескольких инстансов smarty рекомендуется вместо использования
файла настроек ``prod.py`` создать собственный файл с кратким символическим именем, совпадающим с названием сервиса,
например ``myiptv.py``.

Данное имя затем также рекомендуется использование как суффикс или префикс в именах файлов конфигурации *nginx*, *uwsgi*,
именах папок для логов, pid-файлов и др.

.. _settings-description:

Описание основных параметров конфигурации
-----------------------------------------

DEBUG ``bool``
  Включить режим отладки. При возникновении ошибок будут показаны развернутые stacktrace ошибки.

DATABASES ``dict``
  В этом массиве настраивается подключение к СУБД. При использовании нескольких СУБД для репликации необходимо отдельно
  настроить подключение к каждой из них, задав для каждого подключения уникальное имя.

TIME_ZONE ``str``
  Часовой пояс сервера согласно именованию IANA.

SECRET_KEY ``str``
  Уникальный секретный ключ инстанса. Используется для безопасного кеширования секретных данных в процессе работы.
  Не забудьте поменять параметр SECRET_KEY на уникальное значение.

INSTALLED_APPS ``list``
  Список установленных модулей. Список дополняет базовые модули, прописанные в ``base.py``, через операцию ``+=``.

DEFAULT_FROM_EMAIL ``str``
  Отображаемый адрес "От" в отправляемых служебных email по-умолчанию.

EMAIL_FROM ``str``
  Фактически адрес "От" в отправляемых служебных email.

EMAIL_HOST ``str``
  Адрес smtp-сервера.

EMAIL_PORT ``int``
  Порт smtp-сервера.

EMAIL_HOST_USER ``str``
  Имя пользователя для авторизации на smtp-сервере.

EMAIL_HOST_PASSWORD ``str``
  Пароль для авторизации на smtp-сервере.

EMAIL_USE_TLS ``bool``
  Включает использование защищенного протокола TLS при отправке писем.

TVMIDDLEWARE_API_URL ``str``
  URL к интерфейсу TVMiddleware API для использования в клиентских приложениях.
  По-умолчанию "http://smarty.example.com/tvmiddleware/api".

SMARTY_URL ``str``
  URL к инстансу smarty. Используется в панели администратора и в ответах API в качестве базового пути к изображениям и статическим ресурсам.
  По-умолчанию "http://smarty.example.com".

.. _license-settings:

Добавление лицензионного ключа сервера
--------------------------------------

Каждый инстанс smarty привязывается к аппаратной и программной конфигурации сервера лицензионным ключом, который
может быть ограничен по времени действия и максимальному числу настроенных сервисов (см. :ref:`Мультипровайдер <multiprovider>`).

Лицензионный ключ настраивается в файле конфигурации в следующих переменных:
::
    SMARTY_KEY = 'XXXXXXX-XXXXXXX-XXXXXXX-XXXXXXX'
    SMARTY_MAX_CLIENTS = 2
    SMARTY_AVAILABLE_UNTIL = 'dd.mm.yyyy'

Для получения ключа необходимо обратиться к своему менеджеру или отправить письмо на адрес request@microimpuls.com,
сопроводив запрос информацией об имеющемся договоре или гарантийном письме.

.. _cache-settings:

Настройка кеширования
---------------------

Для кеширования используется сервер **Redis**.
Для включения механизма кеширования необходимо в базовом конфигурационном файле ``base.py`` включить опцию **REDIS_ENABLED**.

По-умолчанию конфигурация подразумевает локальную установку сервера Redis на тот же сервер smarty,
однако при необходимости их можно разделить.
Для изменения параметров подключения к Redis необходимо в конфигурации smarty прописать массив **CACHEOPS_REDIS** следующим образом:
::
    CACHEOPS_REDIS = {
        'host': 'X.X.X.X',
        'port': 6379,
        'db': 2, # SELECT non-default redis database using separate redis db or redis instance is highly recommended
        'socket_timeout': 3, # connection timeout in seconds, optional
    }

.. _geo-settings:

Настройка геолокации
--------------------

Поддерживается несколько локаторов на основе IP-адреса абонента, работающие с разными источниками гео-данных.
В :ref:`служебной панели администрирования <service_configuration>` для настраиваемого клиента необходимо установить
используемый локатор, наиболее подходящий для клиента и его региона оказания услуг.

*Если до изменения локатора база данных стран и городов уже была заполнена, то рекомендуется очистить её.*

Все локаторы требуют создания/обновления своей базы данных. База данных может быть в виде SQL-таблиц или бинарных данных (либо и то, и то).

.. _django-geoip:

django-geoip (ipgeobase)
~~~~~~~~~~~~~~~~~~~~~~~~

Обёртка вокруг https://django-geoip.readthedocs.org/en/latest/

Обновление базы:
::
    $ python manage.py geoip_update

Создание стран и городов на основе данных django-geoip (работает только если в системе нет ни одной страны и города):
::
    $ python manage.py sync_geo_geoip

.. _ip2location:

ip2location
~~~~~~~~~~~

Обновление базы:
::
    $ python manage.py update_ip2location

Эта команда скачивает бинарную базу данных для определения местоположения и csv-базу для создания справочника городов и стран.

Создание стран и городов на основе данных ip2location (работает только если в системе нет ни одной страны и города):
::
    $ python manage.py sync_geo_ip2location


После выбора локатора и синхронизации данных механизм геолокации готов к использованию. Доступность тех или иных
сервисов Middleware (телеканалы, фильмы, видеосервисы, опции и т.д.) определяется тарифными планами (см. :ref:`Тарифные планы <admin-tariffs>`),
в настройках которых можно указать те страны и города, в которых они действуют.

.. _monitoring-settings:

Настройка модуля мониторинга видеопотоков
-----------------------------------------

Настройки задаются переменными в файле конфигурации smarty.

MONITORING_CONCURRENT_STREAMS_COUNT ``int``
  Количество одновременно опрашиваемых потоков с анализаторов MicroTS.
  Данный параметр влияет на производительность и быстроту обновления данных на странице мониторинга потоков.

MONITORING_AGENT_SOCKET_TCP_BUFFER ``int``
  Размер буфера данных при приеме ответа от анализатора MicroTS. Значение по-умолчанию 4096 байт.

Скрипт опроса анализаторов запускается через crontab - см. :ref:`Настройка выполнения команд в crontab <crontab-settings>`.

.. _reports-settings:

Настройка модуля статистики и отчетов
-------------------------------------

Для сохранения данных работы модуля статистики и отчетов используется сервер **MongoDB**.
Настройки задаются переменными в файле конфигурации smarty.

MONGODB_HOST ``str``
  Адрес сервера MongoDB.

MONGODB_PORT ``int``
  Порт сервера MongoDB.

MONGODB_NAME ``str``
  Название базы данных.

MONGODB_USERNAME ``str``
  Имя пользователя для авторизации.

MONGODB_PASSWORD ``str``
  Пароль для авторизации.

.. _sms-settings:

Настройка модуля отправки информационных SMS клиентам
-----------------------------------------------------

SMS отправляются системой при использовании виджетов, интегрированных с сайтом, например, во время регистрации абонента.
Настройки задаются переменными в файле конфигурации smarty.

SMS_BACKED ``str``
  Используемый СМС-шлюз для отправки сообщений.
  Модуль, реализующий взаимодействие со шлюзом, должен располагаться в директории smarty в папке ``sms/backends/``.

SMS_ATTEMPTS ``int``
  Количество максимальных попыток отправки сообщения, после которого оно считается отправленным неуспешно.

.. _smsc.ru:

Шлюз smsc.ru
~~~~~~~~~~~~

Значение для **SMS_BACKEND** = ``sms.backends.smscru.SMSCBackend``

SMSC_LOGIN ``str``
  Имя пользователя в сервисе smsc.ru

SMSC_PASSWORD ``str``
  Пароль в сервисе smsc.ru

SMSC_SENDER ``str``
  Имя отправителя, которое будет отображаться в SMS, отправленных через сервис smsc.ru

.. _sentry-settings:

Подключение системы мониторинга ошибок Sentry
---------------------------------------------

Для подключения ``Sentry`` необходимо в файле конфигурации smarty добавить в **INSTALLED_APPS** модуль ``raven.contrib.django.raven_compat``
и прописать параметры подключения:
::
    RAVEN_CONFIG = {
        'dsn': 'http://<LOGIN>:<PASS>@<SENTRY HOST>/<PROJECT>',
    }

Строку подключения можно получить из настроек проекта в Sentry.

.. _logging-settings:

Настройка расширенного логирования
----------------------------------

При необходимости расширенной отладки smarty можно включить логирование всех запросов к Back-end в консоль и запуск в foreground-режиме.
Для этого в конфигурации smarty необходимо добавить опцию **LOGGING** и настроить формат вывода:
::
    LOGGING = {
        'version': 1,
        'disable_existing_loggers': False,
        'formatters': {
            'verbose': {
                'format': '[%(asctime)s] %(levelname)s %(name)s %(lineno)d "%(message)s"'
            },
            'simple': {
                'format': '%(levelname)s %(message)s'
            },
        },
        'filters': {
            'require_debug_false': {
                '()': 'django.utils.log.RequireDebugFalse'
        }
        },
        'handlers': {
            'console': {
                'level': 'DEBUG',
                'class': 'logging.StreamHandler',
                'formatter': 'verbose',
            },
        },
        'loggers': {
            'django': {
                'handlers': ['console'],
                'level': 'DEBUG',
                'propagate': True,
            },
            'django.db.backends': {
                'handlers': ['console'],
                'level': 'DEBUG',
            },
        }
    }

После этого запустить smarty на базе встроенного http-сервера без uwsgi:
::
    $ python /usr/share/nginx/html/microimpuls/smarty/manage.py runserver 0.0.0.0:80 --settings=settings.<settings name>

.. _init-script:

Запуск, остановка, перезапуск процессов smarty
==============================================

Для управления процессами smarty используется init-скрипт ``/etc/init.d/uwsgi``:
::
    $ /etc/init.d/uwsgi
    Usage: /etc/init.d/uwsgi {start|stop|status|restart|reload|force-reload}

Все команды, кроме ``reload``, действуют на все запущенные инстансы smarty.

Логи по-умолчанию сохраняются в ``/var/log/uwsgi/`` и в ``/var/log/nginx/``.

.. _crontab-settings:

Настройка выполнения команд в crontab
=====================================

.. _epg-import-command:

Импорт EPG
----------

Команда:
::
    python /usr/local/nginx/html/microimpuls/smarty/manage.py epg_import --settings=settings.<settings name>
Рекомендуется запускать импорт несколько раз в день для поддержания актуальности телепрограммы
(см. :ref:`Настройка EPG и телеканалов <epg-setup>`).

.. _check-accounts-command:

Команда списания/продления аккаунтов с помощью встроенного биллинга согласно рассчетным периодам
------------------------------------------------------------------------------------------------

Команда:
::
    python /usr/local/nginx/html/microimpuls/smarty/manage.py check_accounts --settings=settings.<settings name>
Рекомендуется запускать каждую ночь
(см. :ref:`Описание встроенного биллинга <builtin-billing>`).

.. _check-streams-command:

Опрос анализаторов TS-потоков MicroTS (модуль мониторинга видеопотоков)
-----------------------------------------------------------------------

Команда:
::
    python /usr/local/nginx/html/microimpuls/smarty/manage.py check_streams --settings=settings.<settings name>
Рекомендуется запускать каждые 1-5 минут для актуального состояния данных.

.. _check-events-command:

Выполнение действий по триггерам модуля мониторинга видеопотоков
----------------------------------------------------------------

Команда:
::
    python /usr/local/nginx/html/microimpuls/smarty/manage.py check_events --settings=settings.<settings name>
Рекомендуется запускать каждую минуту для актуального информирования об авариях.

.. _send-activation-expires-messages-command:

Рассылка информационных сообщений на экраны устройств и email о приближении срока деактивации/необходимости оплаты
------------------------------------------------------------------------------------------------------------------

Команда:
::
    python /usr/local/nginx/html/microimpuls/smarty/manage.py send_activation_expires_messages --days_count <количество оставшихся дней> --settings=settings.<settings name>
Рекомендуется запускать каждый вечер.

.. _clean-old-messages-command:

Очистка старых недоставленных информационных сообщений
------------------------------------------------------

Команда:
::
    python /usr/local/nginx/html/microimpuls/smarty/manage.py clean_old_messages --days_count 3 --settings=settings.<settings name>

.. _resend-sms-command:

Повторная отправка SMS-сообщений, недоставленных с первого раза
---------------------------------------------------------------

Команда:
::
    python /usr/local/nginx/html/microimpuls/smarty/manage.py resend_sms --settings=settings.<settings name>
Рекомендуется запускать каждые 1-3 минуты.

.. _crontab-example:

Пример настройки crontab
------------------------

Пример:
::
    0 5,9,13 * * *      python /usr/local/nginx/html/microimpuls/smarty/manage.py epg_import
    */2 * * * *	        python /usr/local/nginx/html/microimpuls/smarty/manage.py check_streams
    */1 * * * *	        python /usr/local/nginx/html/microimpuls/smarty/manage.py check_events
    0 4 * * *           python /usr/local/nginx/html/microimpuls/smarty/manage.py check_accounts
    0 18 * * *	        python /usr/local/nginx/html/microimpuls/smarty/manage.py send_activation_expires_messages --days_count 5
    0 18 * * *          python /usr/local/nginx/html/microimpuls/smarty/manage.py send_activation_expires_messages --days_count 3
    0 18 * * *          python /usr/local/nginx/html/microimpuls/smarty/manage.py send_activation_expires_messages --days_count 1
    */3 * * * *         python /usr/local/nginx/html/microimpuls/smarty/manage.py resend_sms
    0 3 * * *           python /usr/local/nginx/html/microimpuls/smarty/manage.py clean_old_messages --days_count 3

.. _scalability-failsafe:

Масштабирование и отказоустойчивость
====================================

.. _known-scalability-failsafe-tools:

Доступные к применению средства масштабирования и отказоустойчивости
--------------------------------------------------------------------

Основной способ горизонтального масштабирования **на уровне серверов** smarty заключается в использовании распределенных по серверам
инстансов и распределения запросов между ними на http-сервере nginx через механизм `upstream <http://nginx.org/ru/docs/http/ngx_http_upstream_module.html>`_,
при использовании общего кластера СУБД.

Дополнительной мерой для распределения нагрузки может быть выделение роли инстансов под конкретные цели, например, отделение
сервера для модуля статистики и отчетов и сервера мониторинга от серверов TV Middleware.

Для отказоустойчивости серверов Back-end можно использовать решение **keepalived**, аппаратный балансировщик запросов, механизм Geo DNS или Anycast.

Абонентские приложения (для Set-Top Box, Smart TV и других устройств) могут загружать абонентский портал с отдельных
Front-end серверов, с CDN, либо напрямую из прошивки устройства (т.н. *толстый* клиент), поскольку состоят только из
статических файлов или разработаны на нативном языке устройства. С серверами Back-end абонентские приложения взаимодействуют
через JSON или XML-RPC API.

Кроме того, абонентские приложения имеют встроенный механизм работы с несколькими Back-end серверами, что дополнительно
обеспечивает распределение нагрузки и отказоустойчивость **на уровне приложения**.

Масштабирование и отказоустойчивость **на уровне данных** могут быть обеспечены благодаря механизму репликации СУБД (см. :ref:`далее <database-replication-settings>`).
Поддерживаются схемы Multi-Master и Master-Slave.

.. _database-replication-settings:

Настройка подключения к СУБД с использованием репликации
--------------------------------------------------------

При использовании репликации на уровне СУБД необходимо в файле конфигурации smarty прописать подключение к каждой СУБД
в переменной **DATABASES**, задав каждому подключению уникальное имя.

После этого необходимо добавить в файл конфигурации следующие опции:

DATABASE_SLAVES ``list``
  По-умолчанию, роль каждого подключения определена как *Master*, для выделения *Slave* ролей необходимо
  в массиве DATABASE_SLAVES указать имена подключений, которые будут использоваться как Slave.

DATABASE_ROUTERS ``list``
  Механизм репликации. Используйте встроенный ``django_replicated.ReplicationRouter``.

DATABASE_DOWNTIME ``int``
  Время недоступности сервера БД в секундах, по прошествию которого он отключается из схемы распределения запросов.

Пример:
::
    DATABASE_SLAVES = ['slave1', 'slave2']
    DATABASE_ROUTERS = ['django_replicated.ReplicationRouter']
    DATABASE_DOWNTIME = 60
