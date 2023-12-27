## Полезные ссылки:

### [Ссылка на дашборд в Power BI](https://app.powerbi.com/groups/76c4a426-c01b-440d-94cb-ec61e8209060/reports/de1ae1e5-e7c4-4c6e-85ec-960abad365b8/ReportSection?experience=power-bi&clientSideAuth=0&bookmarkGuid=Bookmark4e63dea321de231355d8)
### [Ссылка на репозиторий](https://github.com/DigitalStrategyMSK/internal_marketing/tree/6d8568a01b75a6afe4b379545c16127286edad64/social_stats)
## Состав проекта и описание файлов:
Представленный проект позволяет собирать данные из различных социальных сетей компании, в основном это данные по подписчикам, просмотрам/охватам и даты последних постов.
Основная логика разделена на 2 части - это файлы main_load.py и meta_load.py.
### [main_load.py](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/social_stats/main_load.py)
Основной функционал и особенности:
1. Запускает функции по выгрузке статистики из соц сетей. В основном используются доступные API кабинетов или парсинг, без использования прокси. 
2. Здесь собраны кабинеты которые доступны на территории РФ.
3. После получения данных, происходит выгрузка в базу данных BigQuery, по каждому кабинету в отдельную таблицу.
4. Функции, которые используются в этой логике, прописаны в следующих файлах:[vk_stats_func.py](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/social_stats/vk_stats_func.py), [dzen_load.py](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/social_stats/dzen_load.py),  [youtube_load.py](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/social_stats/youtube_load.py),  [tg_load.py](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/social_stats/tg_load.py), [tenchat_load.py](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/social_stats/tenchat_load.py), [vc_load.py](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/social_stats/vc_load.py). Соответствуют каждому кабинету.
### [meta_load.py](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/social_stats/meta_load.py)
Файл для выгрузки статистики из Instagram и FaceBook
1. Выгружаются данные из кабинетов и загружаются в таблицу BigQuery. Используется прокси для сбора статистики
2. Статистика собирается из 4 кабинетов компании DS, а именно dsteam_en, dsteam_ru, multicore, idenzy
3. Функции которые используются для выгрузки статистики хранятся в [meta_func.py](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/social_stats/meta_func.py)

Остальные файлы используются для получения разрешений к API сервисов.
