## Полезные ссылки:

### [Ссылка на основное ТЗ по проекту](https://docs.google.com/document/d/1YfpNP16T1Gr1QpKF55hnxoiLwkJ_Q-BnLsl7Ary0nME/edit#heading=h.tgxw3tk6wp9b)
### [Ссылка на репозиторий](https://github.com/DigitalStrategyMSK/drop_catcher)
### [Ссылка на самого бота в Telegram](https://t.me/drop_catcher_bot)

## Состав проекта и описание файлов:

### [tg_bot.py](https://github.com/DigitalStrategyMSK/drop_catcher/blob/main/tg_bot.py)
Главная точка входа в проект. Основной функционал и особенности:
1. Позволяет выполнять команду загрузики в систему своего списка доменов через файл Spreadsheets, указывая информацию по доменам в виде тегов, региона и описания.
   Команда называется `Добавить файл с доменами для сбора данных whois`. Данный функционал выполняет алгоритм прописанный в [whois.py](https://github.com/DigitalStrategyMSK/drop_catcher/blob/main/whois.py)
2. Позволяет выполнять команду сбора метрик по доменам из холда по последней ежедневной проверке.
   Команда называется `Собрать доменные метрики для доменов в холде из последнего файла ежедневных проверок`. Данный функционал выполняет алгоритм прописанный в [get_metrics.py](https://github.com/DigitalStrategyMSK/drop_catcher/blob/main/get_metrics.py)

### [whois.py](https://github.com/DigitalStrategyMSK/drop_catcher/blob/main/whois.py)
Файл для работы с данными из whois протокола. Реализован полный ETL таких данных для доменов из `.com` и `.ru` зон. Всего несколько этапов работы с файлом доменов:
1. Передача доменов в программу как датафрейма
2. Выгрузка данных whois по домену
3. Работа с полученными датами
4. Загрузка полученных в рамках работы программы данных в файл Spreadsheets и базу данных MySQL

### [get_metrics.py](https://github.com/DigitalStrategyMSK/drop_catcher/blob/main/get_metrics.py)
Файл служит для получения доменных метрик с сервиса checktrust.ru и загрузки в файл последней ежедневной проверки

### [daily_check.py](https://github.com/DigitalStrategyMSK/drop_catcher/blob/main/daily_check.py)
Файл ежедневной проверки по доменам в базе данных. Основной функционал:
1. Проверка доменов в базе на предмет продления регистрации или ее окончания
2. Перенос в отдельные таблицы каждоый категории доменов
3. Отправка файла с проверкой на почту

Есть 3 категории доменов:
1. expired таблица - домены, у которых срок регистрации закончится через 5 дней
2. hold таблица - домены в периоде преимущественного продления
3. free таблица - свободные домены

### [db_commands.txt](https://github.com/DigitalStrategyMSK/drop_catcher/blob/main/db_commands.txt)
Команды по созданию всех таблиц в MySQL

### [mysql_lib.py](https://github.com/DigitalStrategyMSK/drop_catcher/blob/main/mysql_lib.py)
Файл для работы с СУБД MySQL

### [sheets_lib.py](https://github.com/DigitalStrategyMSK/drop_catcher/blob/main/sheets_lib.py)
Файл для работы с API Spreadsheets

### [site_parser.py](https://github.com/DigitalStrategyMSK/drop_catcher/blob/main/site_parser.py)
Парсер html страниц сайтов. Собирает их внутри каталога `file_load`
