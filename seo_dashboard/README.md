## Полезные ссылки:

### [Ссылка на дашборд в Power BI](https://app.powerbi.com/groups/76c4a426-c01b-440d-94cb-ec61e8209060/reports/6dd6fab8-2cdc-4e88-a4a4-45441abd14be/ReportSection?experience=power-bi&clientSideAuth=0)
### [Ссылка на репозиторий сбора Трафика](https://github.com/DigitalStrategyMSK/internal_marketing/tree/main/metrika_seo)
### [Ссылка на репозиторий сбора Позиций](https://github.com/DigitalStrategyMSK/internal_marketing/tree/main/spiders_seo)
### [Ссылка на репозиторий GSC](https://github.com/DigitalStrategyMSK/internal_marketing/tree/main/google_search_console)
### [Ссылка на репозиторий Webmaster](https://github.com/DigitalStrategyMSK/internal_marketing/tree/main/webmaster)

## Состав проекта и описание файлов:
В данном проекте выгружаются основные SEO параметры и в дальнейшем отображаются на дашборде в Power BI.
Отслеживаются позиции ключевых запросов из проверок в spiders, показы/клики из GSC, диагностика и другие параметры из Вебмастера, а также трафик по каждому из кабинетов DS.

## Сбор трафика
### [main_seo.py](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/metrika_seo/main_seo.py)

В данном разделе происходит подключение к API Яндекс метрики с помощью токена, определяется период выгрузки и формируется тело запроса, по которым будут грузиться метрики.
Пример кода параметров, которые отсылаются к api:
```python
atribut='lastsign'
            
params = dict(
	id = counter_id,
	lang="ru",
	metrics="ym:s:users",
	filters=f"""ym:s:isRobot=='No'
				AND (ym:s:{atribut}SearchEngineRoot=='yandex' OR ym:s:{atribut}SearchEngineRoot=='Google')""",
				# preset="",
	accuracy="full",
	dimensions=f"ym:s:{atribut}SearchEngineRoot, ym:s:date",
	sort="ym:s:date",
	date1=date_start,
	date2=date_end,
	limit=1000
                        )
```
Таким образом выгружаются посетители из Яндекс Метрики и формируется Датафрейм для выгрузки в базу данных BigQuery. Данные собираются циклом для каждого кабинета DS.

## Сбор позиций

### [load_to_bq.py](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/spiders_seo/load_to_bq.py) - Основная логика
### [data_anal.py](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/spiders_seo/data_anal.py) - Вспомогательные функции

В данным разделе используется api spiders, для выгрузки результатов проверок по позициям и видимости. Код проходится циклом по каждой проверке которая указана в файле [config.ini](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/spiders_seo/config.ini), формирует Датафрейм и выгружает данные в таблицу BigQuery.

## Сбор показателей Google Search Console

### [analit_weekly.py](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/google_search_console/analit_weekly.py) - Основная логика
### [GSC_lab.py](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/google_search_console/GSC_lab.py) - Вспомогательные функции

Сначала определяются доступы к кабинетам GSC Api, для каждого кабинета существует свой json-файл. Данные выгружаются 1 раз в неделю и собираются основные показатели, которые доступны в GSC - показы, CTR, клики, позиции. Полученный результат формируется в Датафрейм и грузится в базу данных BigQuery.

## Сбор показателей Яндекс Вебмастера

### [metr_webmaster.py](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/webmaster/metr_webmaster.py) - Основная логика
### [yandex_parser](https://github.com/DigitalStrategyMSK/internal_marketing/blob/main/webmaster/yandex_parser.py) - Вспомогательные функции

Данные выгружаются из 4 кабинетов компании DS, проект Seocheater выгружается отдельно, так как находится на отдельном аккаунте Вебмастера. Подключение к API Вебмастера происходит по токену, идентификатору кабинета и домену. 
Для каждого проекта собираются основные показатели - диагностика сайта, ответы страниц, динамику индексации, индекс качества сайта, удаленные/добавленные страницы. Полученные данные также выгружаются в BigQuery.