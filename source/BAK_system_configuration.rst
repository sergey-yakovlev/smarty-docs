
**********************
Системная конфигурация
**********************


Системные настройки
===================

Добавление конфигурации
-----------------------

После первичной установки базовый файл конфигурации smarty находится по адресу ``/etc/smarty/base.py``
(симлинк на ``/usr/share/nginx/html/microimpuls/smarty/settings/base.py``).

Основной файл конфигурации, используемый для production-режима работы - ``/etc/smarty/prod.py``.
На этот файл (или на другой используемый конфиг) должен указывать симлинк в ``/usr/share/nginx/html/microimpuls/smarty/settings/<setings name>.py``.
Именно в нем следует производить настройку smarty.

Конфигурация производится путем присваивания значений переменым на Python.

Конфигурация для **nginx** находится в файле ``/etc/nginx/sites.available/smarty``.

Конфигурация для **uwsgi** находится в файле ``/etc/microimpuls/smarty/uwsgi/smarty.uwsgi``,
на него (или на другой используемый конфиг) должен указывать симлинк в ``/usr/share/nginx/html/microimpuls/smarty/<uwsgi settings name>.uwsgi``.

Идеология инстансов
~~~~~~~~~~~~~~~~~~~

Для удобства конфигурации и размещения на одном сервере нескольких инстансов smarty рекомендуется вместо использования
файла настроек ``prod.py`` создать собственный файл с кратким символическим именем, совпадающим с названием сервиса,
например ``myiptv.py``.

Данное имя затем также рекомендуется использование как суффикс или префикс в именах файлов конфигурации *nginx*, *uwsgi*,
именах папок для логов, pid-файлов и др.

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

.. DEFAULT_FROM_EMAIL = 'robot@microimpuls.com'
SERVER_EMAIL = 'robot@microimpuls.com'
EMAIL_FROM = 'robot@microimpuls.com'
EMAIL_HOST = 'smtp.yandex.ru'
EMAIL_PORT = 587
EMAIL_HOST_USER = 'robot@microimpuls.com'
EMAIL_HOST_PASSWORD = ''
EMAIL_USE_TLS = True

.. MAIL_SPAWNER_DOMAINS = ['example.com',]

.. MONITORING_THUMBNAILS_ROOT = rel('media/thumbnails')
MONITORING_THUMBNAILS_URL = '/media/thumbnails/'
MONITORING_CONCURRENT_STREAMS_COUNT = 15
MONITORING_STREAM_TEST_DURATION = 10
MONITORING_AGENT_SOCKET_TCP_BUFFER = 4096

.. TVMIDDLEWARE_PORTAL_DOMAIN = 'microimpuls.com'
TVMIDDLEWARE_API_URL = 'http://smarty.microimpuls.com/tvmiddleware/api'
SMARTY_URL = 'http://smarty.microimpuls.com'

.. SMS_BACKEND = 'sms.backends.smscru.SMSCBackend'
SMS_ATTEMPTS = 3

.. SMSC_LOGIN = 'microimpuls'
SMSC_PASSWORD = ''
SMSC_SENDER = 'ImpulsTV'

.. MONGODB_HOST = 'mdb1.vm2.noris.nbg.mpls.im'
MONGODB_PORT = 27017
MONGODB_NAME = 'smarty_microimpuls'
MONGODB_USERNAME = 'smarty_microimpuls_admin'
MONGODB_PASSWORD = ''

.. DATABASE_SLAVES = ['slave1', 'slave2']
DATABASE_SLAVES = []
DATABASE_ROUTERS = ['django_replicated.ReplicationRouter']
DATABASE_DOWNTIME = 60

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


Настройка кеширования
---------------------

Настройка геолокации
--------------------

Настройка модуля мониторинга видеопотоков
-----------------------------------------

Настройка модуля статистики и отчетов
-------------------------------------

Настройка модуля отправки информационных СМС клиентам
-----------------------------------------------------


Подключение системы мониторинга ошибок Sentry
---------------------------------------------

добавить в модули raven.contrib.django.raven_compat
и добавить в settings:
RAVEN_CONFIG = {
   'dsn': 'http://33c7020cb60340f1a2dfca25e4f7f188:0a2b2cee6ede45c2a7aac33d3e35eecc@sentry.mpls.im/5',
}

Настройка расширенного логирования
----------------------------------

"""
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
"""

Запуск, остановка, перезапуск
=============================

Для управления процессами smarty используется init-скрипт ``/etc/init.d/uwsgi``:
::
    # /etc/init.d/uwsgi
    Usage: /etc/init.d/uwsgi {start|stop|status|restart|reload|force-reload}

Все команды, кроме ``reload``, действуют на все запущенные инстансы smarty.

Логи по-умолчанию сохраняются в ``/var/log/uwsgi/`` и в ``/var/log/nginx/``.


Настройка выполнения команд в crontab
=====================================


Масштабирование и отказоустойчивость
====================================

Доступные к применению средства масштабирования и отказоустойчивости
--------------------------------------------------------------------

Настройка подключения к СУБД с использованием репликации
--------------------------------------------------------

