.. _log:

****************************
4. Логирование работы Smarty
****************************

4.1. Логи Smarty
================

Записи в логах имеют следующий вид: ::

    %(timestamp)% %(log level)% %(module)%:%(code line number)% %(function)%[%(pid)%.%(thread id)%] %(message text)%

Сообщение (message text) состоит из названия события и дополнительного контекста.

Пример записи в логе: ::

    Fri Mar 24 10:22:35+0300 2017 DEBUG api:621 get[83630.Thread-3] SOME_EVENT 'ip'='127.0.0.1' args=['foo', 'bar']

В этом примере:

Fri
    день недели
Mar
    месяц
24
    дата
10:22:35
    локальное время в таймзоне, которая указана в settings.py в константе TIME_ZONE
+0300
    смещение относительно UTC
2017
    год
DEBUG
    уровень лога
api:621
    название файла и номер строки
get[83630.Thread-3]
    название функции, pid и имя треда
SOME_EVENT
    название события
*ip* и *args*
    дополнительный контекст


Уровень лога выстваляется в настройках проекта(smarty/settings/settings_<local>.py).
Например, чтобы выставить уровень INFO для лога запросов к api, необходимо добавить следующий код:  ::

    LOGGING['handlers']['smarty_api_requests_handler']['level'] = "INFO"


События в зависимости от типа группируются в разных файлах. По умолчанию, логи сохраняются по адресу ``/var/log/microimpuls/smarty/``.

4.1.1. smarty_main - основной лог
---------------------------------

Типы событий:

URLS_INITIALIZED
++++++++++++++++

Конфигурация Smarty инициализирована.
Происходит во время старта приложения, дополнительный контекст:

* ``available_apps`` - доступные модули


События, необработанные другими логгерами
+++++++++++++++++++++++++++++++++++++++++

Происходит во время ошибки, необходимо обратиться к разработчику для решения проблемы.

Дополнительный контекст: локальные переменные функции, в которой произошла ошибка.


4.1.2. smarty_accounts - лог абонентов
--------------------------------------------------------

Типы событий:

LOGIN_SUCCESS
+++++++++++++

Успешная авторизация, контекст:

* ``login_type`` - тип логина (мультилогин, базовое устройство, дополнительное устройство)
* ``login_method`` - метод логина (по абонементу и паролю, только по абонементу, только по UID устройства)
* ``params`` - параметры запроса


LOGIN_ERROR
+++++++++++

Ошибка авторизации, контекст:

* ``reason`` - причина ошибки:
* * account_does_not_exist - аккаунт не существует - абонемент или пароль или device_uid неверный
* * account_is_not_active - аккаунт не активен
* * client_does_not_exist - неверный client_id или api_key
* * basic_device_sessions_limit - превышен лимит сессий на базовом устройстве
* * account_deivce_invalid - такое базовое устройство уже существует
* * external_api_error - ошибка запроса к внешней системе (например, к внешнему биллингу)
* * unknown_error - неизвестная ошибка (сопровождается stacktrace'ом)
* ``params`` - параметры запроса


ACCOUNT_DEVICE_ERROR
++++++++++++++++++++

Ошибка при создании устройства аккаунта, контекст:

* ``account`` - аккаунт
* ``device`` - устройство
* ``device_uid`` - UID устройства
* ``client`` - оператор

ACCOUNT_ACTIVATED
+++++++++++++++++

Аккаунт был активирован, контекст:

* ``activation_result`` - ответ, который вернула внешняя система
* ``params`` - параметры запроса



ACCOUNT_CREATED
+++++++++++++++

Создание аккаунта, контекст:

* ``account_id`` - идентификатор аккаунта


ACCOUNT_CHANGED
+++++++++++++++

Изменение аккаунта, контекст:

* ``account_id`` - идентификатор аккаунта
* изменённые поля



ACCOUNT_TARIFFS_ASSIGNED
++++++++++++++++++++++++

Добавление тарифов аккаунту, контекст:

* ``account_id`` - идентификатор аккаунта
* ``tariffs_ids`` - список идентификаторов подключенных тарифов


ACCOUNT_TARIFFS_REMOVED
+++++++++++++++++++++++

Удаление тарифов у аккаунта, контекст:

* ``account_id`` - идентификатор аккаунта
* ``tariffs_ids`` - список идентификаторов удаленных тарифов


CUSTOMER_CREATED
++++++++++++++++

Создание абонента, контекст:

* ``customer_id`` - идентификатор абонента


CUSTOMER_CHANGED
++++++++++++++++

Изменение абонента, контекст:

* ``customer_id`` - идентификатор абонента
* изменённые поля


CUSTOMER_TARIFFS_ASSIGNED
+++++++++++++++++++++++++

Добавление тарифов абоненту, контекст:

* ``customer_id`` - идентификатор абонента
* ``tariffs_ids`` - список идентификаторов подключенных тарифов


CUSTOMER_TARIFFS_REMOVED
++++++++++++++++++++++++

Удаление тарифов у абонента, контекст:

* ``customer_id`` - идентификатор абонента
* ``tariffs_ids`` - список идентификаторов удаленных тарифов


ACCOUNT_DEVICE_REMOVED
++++++++++++++++++++++

Удаление устройства аккаунта, контекст:

* ``account_id`` - идентификатор аккаунта
* ``device_uid`` - идентификатор устройства


4.1.3. smarty_billing_out - запросы к внешним системам
------------------------------------------------------

INIT_ERROR
++++++++++

Ошибка инициализации обработчика API, контекст:

* ``kwargs`` - аргументы, переданные в класс обработчика
* ``api_client_class`` - название класса обработчика API
* stacktrace


CUSTOMER_BALANCE_REQUEST_ERROR
++++++++++++++++++++++++++++++

Ошибка при запросе баланса, контекст:

* ``api_client_class`` - название класса обработчика API
* ``params`` - параметры запроса
* stacktrace


CUSTOMER_BALANCE_REQUEST_SUCCESS
++++++++++++++++++++++++++++++++

Успешный запрос баланса, контекст:

* ``api_client_class`` - название класса обработчика API
* ``params`` - параметры запроса
* ``result`` - результат запроса


CUSTOMER_PAYMENT_LIST_REQUEST_ERROR
+++++++++++++++++++++++++++++++++++

Ошибка при запросе списка транзакций, контекст:

* ``api_client_class`` - название класса обработчика API
* ``params`` - параметры запроса
* stacktrace


CUSTOMER_PAYMENT_LIST_REQUEST_SUCCESS
+++++++++++++++++++++++++++++++++++++

Успешный запрос списка транзакций, контекст:

* ``api_client_class`` - название класса обработчика API
* ``params`` - параметры запроса
* result - результат запроса


VIDEO_ACTIONS_LIST_REQUEST_ERROR
++++++++++++++++++++++++++++++++

Ошибка при запросе вариантов действий с видео, контекст:

* ``api_client_class`` - название класса обработчика API
* ``params`` - параметры запроса
* stacktrace


VIDEO_ACTIONS_LIST_REQUEST_SUCCESS
++++++++++++++++++++++++++++++++++

Успешный запрос вариантов действий с видео, контекст:

* ``api_client_class`` - название класса обработчика API
* ``params`` - параметры запроса
* result - результат запроса


VIDEO_ACTION_REQUEST_ERROR
++++++++++++++++++++++++++

Ошибка при попытке произвести действие с видео, контекст:

* ``api_client_class`` - название класса обработчика API
* ``params`` - параметры запроса
* stacktrace


VIDEO_ACTION_REQUEST_SUCCESS
++++++++++++++++++++++++++++

Успешное действие с видео, контекст:

* ``api_client_class`` - название класса обработчика API
* ``params`` - параметры запроса
* result - результат запроса


4.1.4. smarty_billing_in - входящие запросы к Billing API
---------------------------------------------------------

BILLING_REQUEST_ERROR
+++++++++++++++++++++

Ошибка при запросе к Billing API, контекст:

* ``url`` - URL метода, к которому производился запрос
* ``ip`` - IP-адрес, с которого производлися запрос
* ``args`` - аргументы запроса
* ``error_message`` - сообщение об ошибке
* ``error`` - код ошибки


BILLING_REQUEST_SUCCESS
+++++++++++++++++++++++

Успешный запрос в биллинг, контекст:

* ``url`` - URL метода, к которому производился запрос
* ``ip`` - IP-адрес, с которого производлися запрос
* ``args`` - аргументы запроса


4.1.5. smarty_cache - события, связанные с кешированием
-------------------------------------------------------

OBJECT_CACHED
+++++++++++++

Обьект закеширован, контекст:

* ``object`` - кешируемый обьект
* ``timeout`` - время, по истечении которого обьект будет удален из кеша
* ``key`` - ключ обьекта в кеше
* ``deps`` - обьекты, при изменении которых кешируемый обьект должен быть инвалидирован


OBJECT_INVALIDATED
++++++++++++++++++

Обьект инвалидирован, контекст:

* ``object`` -  обьект, который был удален из кеша
* ``deps_key`` - ключ обьекта, где находятся ключи связанных обьектов, которые тоже должны быть инвалидированы


4.1.6. smarty_messaging - лог отправленных сообщений для аккаунтов
------------------------------------------------------------------

MESSAGE_CREATED
+++++++++++++++

Создано сообщение, контекст:

* ``account`` - аккаунт, которому было отправлено сообщение
* ``subject`` - тема сообщения
* ``text`` - текст сообщения


MESSAGE_SEND
++++++++++++

Сообщение отправлено, контекст:

* ``account`` - аккаунт, которому было отправлено сообщение
* ``subject`` - тема сообщения
* ``text`` - текст сообщения
* ``uuid`` - идентификатор сообщения


MESSAGE_DELETED
+++++++++++++++

Сообщение удалено, дополнительный контекст:

* ``account`` - аккаунт, которому было отправлено сообщение
* ``subject`` - тема сообщения
* ``text`` - текст сообщения
* ``uuid`` - идентификатор сообщения


4.1.7. smarty_management - лог периодических команд
---------------------------------------------------

MANAGEMENT_COMMAND_SUCCESS
++++++++++++++++++++++++++

Успешное выполнение команды, дополнительный контекст:

* ``command`` - название команды
* ``execution_time`` - время выполнения


MANAGEMENT_COMMAND_ERROR
++++++++++++++++++++++++

Ошибка выполнения команды, дополнительный контекст:

* ``command`` - название команды
* stacktrace


4.1.8. smarty_epg - лог импорта EPG
-----------------------------------

EPG_CHANNEL_IMPORTED
++++++++++++++++++++

Программы для канала успешно импортированы, дополнительный контекст:

* ``epg_channel`` - канал
* ``source`` - источник EPG
* ``programs_imported`` - количество импортированных программ


EPG_CHANNEL_IMPORT_ERROR
++++++++++++++++++++++++

Ошибка при импорте программ, дополнительный контекст:

* ``epg_channel`` - канал
* ``source`` - источник EPG
* stacktrace


EPG_IMPORT_FINISHED
+++++++++++++++++++

Импорт программ завершен, дополнительный контекст:

* ``channels_processed`` - количество обработанных каналов
* ``programms_imported`` - количество импортированных программ


EPG_REMOVED
+++++++++++

В ходе парсинга были удалены старые записи, дополнительный контекст:

* ``epg_channel`` - канал
* ``source`` - источник epg
* ``removed_objects`` - удаленные обьекты


EPG_TIME_OVERLAP
++++++++++++++++

Время окончания предыдущей программы больше времени начала текущей, дополнительный контекст:

* ``epg_channel`` - канал
* ``source`` - источник epg
* ``program_name`` - название программы
* ``previous_program_name`` - название предыдущей программы
* ``program_time_begin`` - время начала текущей программы
* ``previous_time_end`` - время окончания предыдущей программы


EPG_TIME_HOLE
+++++++++++++

Время окончания предыдущей программы меньше времени начала текущей, дополнительный контекст:

* ``epg_channel`` - канал
* ``source`` - источник epg
* ``program_name`` - название программы
* ``previous_program_name`` - название предыдущей программы
* ``program_time_begin`` - время начала текущей программы
* ``previous_time_end`` - время окончания предыдущей программы


EPG_NAME_DOUBLE
+++++++++++++++

Название текущей программы совпадает с предыдущей, дополнительный контекст:

* ``epg_channel`` - канал
* ``source`` - источник epg
* ``program_name`` - название программы


4.1.9. smarty_content_requests - запросы на получение ссылки/адреса потока через TVMW API
-----------------------------------------------------------------------------------------

CONTENT_REQUEST_FAIL
++++++++++++++++++++

Произошла необработанная ошибка в процессе запроса, необходимо обратиться к разработчику.

Дополнительный контекст:

* ``url``  - URL метода, к которому производился запрос
* ``params`` - параметры запроса
* stacktrace

CONTENT_REQUEST_ERROR
+++++++++++++++++++++

Обработанная ошибка в процессе запроса, дополнительный контекст:

* ``url``  - URL метода, к которому производился запрос
* ``params`` - параметры запроса

Возможные причины:

* неверные параметры запроса
* устаревший ключ авторизации
* запрос к устравшем данным (например, попытка воспроизвести слишком старую передачу из архива)


CONTENT_REQUEST_SUCCESS
+++++++++++++++++++++++

Успешный запрос, ссылка получена, дополнительный контекст:

* ``url`` - URL метода, к которому производился запрос
* ``params`` - параметры запроса
* дополнительная информация, в т.ч. адрес потока (в зависимости от метода)


CLIENT_CHANNELS_NOT_FOUND
+++++++++++++++++++++++++

В кеше не обнаружены каналы для данного Client ID,
возможно был сброшен кеш или произошла ошибка выполнения команды ``cache_channel_list``, дополнительный контекст:

* ``client`` - Client ID


4.1.10. smarty_portal - лог событий портала
-------------------------------------------

PORTAL_EVENT
++++++++++++

Событие в портале, дополнительный контекст:

* ``event_description`` - описание события
* ``ip`` - IP-адрес устройства абонента
* ``device_uid`` - идентификатор устройства
* ``screen_name`` - название экрана
* ``user_agent`` - User-Agent устройства


4.1.11. smarty_stream_services - лог стриминг-сервисов
-------------------------------------------------------

STREAM_SERVICE_CHECKING_ERROR
+++++++++++++++++++++++++++++

Ошибка при проверке доступности стриминг-сервиса, дополнительный контекст:

* ``stream_service`` - стриминг-сервис
* ``was_available_before`` - указывает, был ли стриминг-сервис доступен ранее
* ``check_ping_success`` - была ли успешной проверка пингом (опционально)
* ``check_tcp_success``- была ли успешной проверка попыткой открыть сокет (опционально)
* ``check_http_success`` - была ли успешной проверка попыткой открыть URL (опционально)
* ``check_is_alive_success`` - была ли успешной проверка is_alive (опционально)
* stacktrace


STREAM_SERVICE_CHECKING_SUCCESS
+++++++++++++++++++++++++++++++

Успешная проверка доступности стриминг-сервиса, дополнительный контекст:

* ``stream_service`` - стриминг-сервис
* ``was_available_before`` - указывает, был ли стриминг-сервис доступен ранее
* ``check_ping_success`` - проверка пингом была успешной (опционально)
* ``check_tcp_success``- проверка попыткой открыть сокет была успешной (опционально)
* ``check_http_success`` - проверка попыткой открыть URL была успешной (опционально)
* ``check_is_alive_success`` - проверка is_alive была успешной (опционально)



4.1.12. smarty_admin - лог панели администрирования Smarty
----------------------------------------------------------

ADMIN_REQUEST
+++++++++++++

Запрос к административному интерфейсу, дополнительный контекст:

* ``user`` - пользователь, осуществивший запрос
* ``ip`` - IP пользовтаеля
* ``path`` - путь запроса
* ``user_agent`` - User-Agent браузера

4.1.13. smarty_videoservices - лог обращений к видеосервисам
------------------------------------------------------------

VIDEOSERVICES_REQUEST
+++++++++++++++++++++

Запрос пользователя в Smarty на выполнение команды:

* ``ip`` - IP пользовтаеля
* ``user_agent`` - User-Agent браузера
* ``user`` - пользователь, осуществивший запрос
* ``path`` - путь запроса

VIDEOSERVICES_API_REQUEST
+++++++++++++++++++++++++

Запрос Smarty к видеосервису:

* ``host`` - тип и адрес видеосервиса
* ``args`` - аргументы запроса
* ``command`` - вызываемый метод
* ``message`` - ответ сервера (только при ошибке)


4.1.14. smarty_payment - лог оплаты
-----------------------------------

NOTIFY_ERROR
++++++++++++

Ошибка обработки сообщения нотификации, дополнительный контекст:

* ``client`` - ID клиента в платёжном шлюзе.
* ``transaction`` - ID транзакции.
* ``payment_source`` - название платёжного шлюза.
* ``params`` - параметры запроса.
* ``error`` - описание ошибки.

Возможные причины ошибки:

* Неверная настройка платёжного шлюза.
* Передача неверных параметров платёжному шлюзу.
* Платёж не прошёл.

NOTIFY_SUCCESS
++++++++++++++

Успешная нотификация, дополнительный контекст:

* ``client`` - ID клиента в платёжном шлюзе.
* ``transaction`` - ID транзакции.
* ``payment_source`` - название платёжного шлюза.
* ``params`` - параметры запроса.

PAYTURE_REQUEST
+++++++++++++++

Запрос к платёжному шлюзу Payture:

* ``url`` - URL API Payture, на который выполняется запрос.
* ``args`` - аргументы запроса.
* ``response`` - ответ от API в виде XML.


