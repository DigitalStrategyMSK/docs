### [Ссылка на репозиторий](https://github.com/DigitalStrategyMSK/DS_internal_projects/tree/pbn_skript/vk_group_stats/new)

## Состав проекта и описание файлов:
Данный скрипт собирает подписчиков соц сетей компании DS и отправляет данные в группу ТГ.

### [vk_new_week.py](https://github.com/DigitalStrategyMSK/DS_internal_projects/blob/pbn_skript/vk_group_stats/new/vk_new_week.py)
Главный файл в котором прописана логика сбора данных по подписчикам. Используется API или же парсинг главных страниц, если api не предоставляется или тяжело получить доступ к api.
Собранные данные выгружаются в Clickhouse, методы для выгрузки данных прописаны в файле [clickhouse.py](https://github.com/DigitalStrategyMSK/DS_internal_projects/blob/pbn_skript/vk_group_stats/new/clickhouse.py). 
Каждую неделю в понедельник производится рассылка собранных данных в группу тг, а также сравниваются результаты по сравнению с прошлой неделей, то есть отслеживается динамика количества подписчиков.

### [get_two_subs.py](https://github.com/DigitalStrategyMSK/DS_internal_projects/blob/pbn_skript/vk_group_stats/new/get_two_subs.py)
Прописан парсинг данных в котором используется авторизация, а также прокси, так как собираются данные из facebook и linkedin.