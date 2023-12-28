Код лежит на сервере 176.112.209.190. Запущен код с помощью сервиса notion_bot.service
## Полезные ссылки:

### Ссылка на бота: https://t.me/notion_task_bot
### [Ссылка на репозиторий](https://github.com/DigitalStrategyMSK/DS_internal_projects/tree/9c9be39344d76b41a56d5b9e33a59e4840a98c8e/notion_bot)

### Notion таблицы: [Чистилище](https://www.notion.so/8a688e98892c4d9e81c4844f937a43ee?v=a44b700110884c58bb66c3891bee60a4), [Список чатов](https://www.notion.so/faeef1f2e9a64ebca19d2d1d81257e94?v=ee30129cde9547fabc1a72e351cd0571), [Список почты gmail](https://www.notion.so/6d46d1ac611943689a255b39c33b77a2?v=26b4ec395fd94e8abaf8e026b91bf762)

# Содержание руководства
### [Этап №1: Импорт библиотек](#step1)
### [Этап №2: Объявление постоянных параметров для работы с Notion](#step2)
### [Этап №3: Объявление основных функций](#step3)
### [Этап №4: Основной функционал при упоминании бота](#step4)
### [Этап №5: Автоматическое заполнение исполнителей](#step5)
### [Этап №6: Выгрузка писем из Gmail](#step6)

## Состав проекта и описание файлов:
При упоминании бота в чате, он обрабатывает сообщение, и выгружает результат в базу данных Notion  - [Чистилище](https://www.notion.so/8a688e98892c4d9e81c4844f937a43ee?v=a44b700110884c58bb66c3891bee60a4)

# Основная логика бота [main.py](https://github.com/DigitalStrategyMSK/DS_internal_projects/blob/pbn_skript/notion_bot/main.py)
<a name="step1"><h3>Этап №1: Импорт библиотек</h3></a>

```python
import configparser
from pyrogram import Client, filters
from datetime import datetime
import requests
```
<a name="step2"><h3>Этап №2: Объявление постоянных параметров для работы с Notion</h3></a>
```python
config = configparser.ConfigParser()
config.read("config.ini")

# Токен аккаунта Notion
NOTION_TOKEN = config['Notion']['notion_token']

# ID базы знаний Notion
DATABASE_ID = '8a688e98892c4d9e81c4844f937a43ee'

# ID таблицы Списка чатов
DATABASE_ID_CHAT = 'faeef1f2e9a64ebca19d2d1d81257e94'

headers = {
    "Authorization": "Bearer " + NOTION_TOKEN,
    "Content-Type": "application/json",
    "Notion-Version": "2022-06-28",
}
```
<a name="step3"><h3>Этап №3: Объявление основных функций</h3></a>
С помощью `post` запроса и словаря `data`, происходит выгрузка сообщения из тг в Notion таблицу - [Чистилище](https://www.notion.so/8a688e98892c4d9e81c4844f937a43ee?v=a44b700110884c58bb66c3891bee60a4)
```python
# Основная функция создания записи в Notion
def create_page(data: dict):
    '''В функцию передается словарь с полями необходимыми для выгрузки
    в базу данных Notion
    '''
    create_url = "https://api.notion.com/v1/pages"
    payload = {"parent": {"database_id": DATABASE_ID}, "properties": data}
    try:
        res = requests.post(create_url, headers=headers, json=payload)
        res.raise_for_status()  # Генерировать исключение, если ответ сервера не успешный
        print("Страница успешно создана в Notion!")
        return res
    except requests.exceptions.HTTPError as err:
        print(f"Ошибка HTTP при создании страницы: {err}")
    except Exception as e:
        print(f"Произошла ошибка: {e}")
```

С помощью `post` запроса происходит загрузка данных в Notion таблицу [Список чатов](https://www.notion.so/faeef1f2e9a64ebca19d2d1d81257e94?v=ee30129cde9547fabc1a72e351cd0571)
```python
# Функция добавления записи в Notion, про информацию чата, в котором был упомянут бот 
def create_chat_list(data: dict):
    create_url = "https://api.notion.com/v1/pages"
    payload = {"parent": {"database_id": DATABASE_ID_CHAT}, "properties": data}

    try:
        res = requests.post(create_url, headers=headers, json=payload)
        res.raise_for_status()  # Генерировать исключение, если ответ сервера не успешный
        print("Страница успешно создана в Notion!")
        return res
    except requests.exceptions.HTTPError as err:
        print(f"Ошибка HTTP при создании страницы: {err}")
    except Exception as e:
        print(f"Произошла ошибка: {e}")
```

Функция используется в случае, если бот в чате упоминается впервые. В таком случае собираются основные данные о чате, и грузятся в таблицу [Список чатов](https://www.notion.so/faeef1f2e9a64ebca19d2d1d81257e94?v=ee30129cde9547fabc1a72e351cd0571)
```python
# Формирует данные о чате в виде словаря. Для публичных и частных групп
def on_chat_member_load(client, message):
    chat = message.chat
    chat_id = chat.id
    chat_private = app.get_chat(chat_id)
    invite_link = chat_private.invite_link
    chat_name = chat.title
    chat_url = f"https://t.me/{chat.username}"
    owner = ""
      
    # Если группа публичная
    if chat.username != None:
        data = {
            "Chat": {"title": [{"text": {"content": chat_name}}]},
            "Chat_id": {"number": chat_id},
            "Chat_url": {"url": chat_url},
            "Owner": {"rich_text": [{"text": {"content": owner}}]}
        }
        if create_chat_list(data):
            print('Запись успешно добавлена!')
    else: # Если группа Частная
        data = {
            "Chat": {"title": [{"text": {"content": chat_name}}]},
            "Chat_id": {"number": chat_id},
            "Chat_url": {"url": invite_link},
            "Owner": {"rich_text": [{"text": {"content": owner}}]}
        }

        if create_chat_list(data):
            print('Запись успешно добавлена!')
```

Триггер срабатывает когда бота добавляют в новый чат. После чего бот собирает данные о чате и грузить в Notion [Список чатов](https://www.notion.so/faeef1f2e9a64ebca19d2d1d81257e94?v=ee30129cde9547fabc1a72e351cd0571)
```python
# Триггер, который обрабатывает информацию о публичном чате и грузит в Notion
@app.on_chat_member_updated()
def on_chat_member_added(client, chat_member_updated):
    if chat_member_updated.old_chat_member != None:
        return print("Кого-то исключили из группы")

    chat = chat_member_updated.chat
    chat_id = chat.id
    chat_name = chat.title
    chat_url = f"https://t.me/{chat.username}"
    owner = ""

    if chat.username != None:
        data = {
            "Chat": {"title": [{"text": {"content": chat_name}}]},
            "Chat_id": {"number": chat_id},
            "Chat_url": {"url": chat_url},
            "Owner": {"rich_text": [{"text": {"content": owner}}]}
        }

        if create_chat_list(data):
            print('Запись успешно добавлена!')
```

<a name="step4"><h3>Этап №4: Основной функционал при упоминании бота</h3></a>

Создаем клиент Telegram
```python
app = Client("my_session")
```

Объявляем декоратор и функцию упоминания бота. Далее с помощью `post` запроса выгружаем данные о чатах в которых уже упоминался бот
```python
@app.on_message(filters.mentioned)
def handle_message(client, message):
    # url для отправки post запроса
    url = f"https://api.notion.com/v1/databases/{DATABASE_ID_CHAT}/query"
    # Выгрузка данных по чатам, в которых упоминался бот
    try:
        res = requests.post(url, headers=headers)
        res.raise_for_status()  # Генерировать исключение, если ответ сервера не успешный
        print("Данные по чатам выгружены")
    except requests.exceptions.HTTPError as err:
        print(f"Ошибка HTTP при создании страницы: {err}")
    data = res.json()
```

Проверяем, упоминался ли бот ранее в чате, если нет, то добавляем чат в таблицу Notion(Список чатов)
```python
    # Объявление списка чатов, в которых уже упоминался бот
    id_list = []
    for i in data['results']:
        number = i['properties']['Chat_id']['number']
        id_list.append(number)
    # Если бот упоминается в чате первый раз
    if message.chat.id not in id_list:
        on_chat_member_load(client, message)
```

Обработка текста сообщения, автора и определение даты. 
Если в сообщении отсутствуют какие либо документы(картинки, видео), то текст будет обрабатываться через метод `message.caption` иначе через `message.text.markdown`
```python
chat_id = client.get_chat(message.chat.id)
chat = chat_id.title
user = client.get_chat_member(message.chat.id, message.from_user.id)
authors = [user.user.username]
char_remove = ['@notion_task_bot', '[', ']']

if message.caption:
	text_url = message.caption.markdown
	for char in char_remove:
		text_url = text_url.replace(char, '').strip()
	text = text_url
else:
	text_url = message.text.markdown
	for char in char_remove:
		text_url = text_url.replace(char, '').strip()
	text = text_url
published_date = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
```

Завершение функции если переслали сообщение от бота, и упомянули его в сообщении. Такие сообщения бот не обрабатывает
```python
if message.reply_to_message and message.reply_to_message.from_user.id == client.me.id:
	return
```

Обработка случая, если сообщение переслали как отвеченное внутри чата.
Объявляем переменную `text`, Проверяем есть ли в ответном сообщении текст и отделяем оба текста сообщений двумя строками `text = text + "\n\n" + text_reply`
```python
if message.reply_to_message:
	reply_user = client.get_chat_member(message.chat.id, message.reply_to_message.from_user.id)
	if (reply_user.user.username not in authors) and reply_user.user.username != 'notion_task_bot':
		authors.append(reply_user.user.username)
	if message.reply_to_message.caption:
		text_reply_url = message.reply_to_message.caption.markdown
		for char in char_remove:
			text_reply_url = text_reply_url.replace(char, '').strip()
		text_reply = '"' + text_reply_url + '"'
		if text == '':
			text = text_reply
		else:
			text = text + "\n\n" + text_reply
	else:
		text_reply_url = message.reply_to_message.text.markdown
		for char in char_remove:
			text_reply_url = text_reply_url.replace(char, '').strip()
		text_reply = '"' + text_reply_url + '"'
		if text == '':
			text = text_reply
		else:
			text = text + "\n\n" + text_reply
```

Если авторов несколько, то разделяем их запятой
```python
# Список авторов при добавлении сообщения с ответом
    author = ", ".join(authors)
```

Формируем тело запроса в json формате
```python
# Оформление данных в json формат
    data = {
        "Chat": {"title": [{"text": {"content": chat}}]},
        "Authors": {"rich_text": [{"text": {"content": author}}]},
        "Published": {"date": {"start": published_date, "end": None}},
        "Status": {
            "type": "select",
            "select": {
                "name": "Новое"
            }
        },
        "Text": {"rich_text": [{"text": {"content": text}}]},
        "Executor": {"rich_text": [{"text": {"content": ""}}]}
}
```

Загружаем сформированные данные `data` в базу данных Notion -  [Чистилище](https://www.notion.so/8a688e98892c4d9e81c4844f937a43ee?v=a44b700110884c58bb66c3891bee60a4). 
При возникновении ошибки, отлавливаем ее и уведомляем об этом в чате.
```python
try:
	if create_page(data):
		link = "[Запись](https://www.notion.so/d445e84a4b784878a16071e3b71d17eb)"
		message.reply_text(
			f"{link} успешно добавлена!")
except Exception as ex:
	message.reply_text(f"При загрузке записи произошла ошибка")
```

Запуск бота
```python
app.run()
```

<a name="step5"><h3>Этап №5: Автоматическое заполнение исполнителей</h3></a>

[notion_script.py](https://github.com/DigitalStrategyMSK/DS_internal_projects/blob/pbn_skript/notion_bot/notion_script.py)
Импорт библиотек
```python
import configparser
from pyrogram import Client, filters
from datetime import datetime
import requests
import re
from logs import logs
```

Импорт токена и идентификаторов таблиц Notion
```python
config = configparser.ConfigParser()
config.read("config.ini")

# Токен аккаунта Notion
NOTION_TOKEN = config['Notion']['notion_token']

# ID таблицы Списка чатов
DATABASE_ID_CHAT = 'faeef1f2e9a64ebca19d2d1d81257e94'

# ID базы знаний Notion
DATABASE_ID = '8a688e98892c4d9e81c4844f937a43ee'
```

Объявление заголовка и url отправки запроса к API. Получение данных из таблицы Notion [Список чатов](https://www.notion.so/faeef1f2e9a64ebca19d2d1d81257e94?v=ee30129cde9547fabc1a72e351cd0571)
```python
# Заголовок запроса для получения ответа по API
headers = {
    "Authorization": "Bearer " + NOTION_TOKEN,
    "Notion-Version": "2022-06-28",
    'Content-Type': 'application/json',
}

url = f"https://api.notion.com/v1/databases/{DATABASE_ID_CHAT}/query"
# Выгрузка данных по чатам, в которых упоминался бот
try:
    res = requests.post(url, headers=headers)
    res.raise_for_status()  # Генерировать исключение, если ответ сервера не успешный
    logs("Данные по чатам получены")
except requests.exceptions.HTTPError as err:
    logs(f"Ошибка HTTP при создании страницы: {err}")
```

Формируем словарь Название чата -> Исполнитель
```python
# Сбор списка чатов и их владельцев
data = res.json()
owner_dict = {}
for item in data["results"]:
    try:
        if item['properties']['Owner']['rich_text'] != []:
            owner = item['properties']['Owner']['rich_text'][0]['text']['content']
            chat_name = item['properties']['Chat']['title'][0]['text']['content']
            owner_dict[f'{chat_name}'] = owner
    except Exception as ex:
        logs(f'Ошибка при обработке чатов: {ex}')
logs(owner_dict)
```

Получение данных из таблицы Notion [Чистилище](https://www.notion.so/8a688e98892c4d9e81c4844f937a43ee?v=a44b700110884c58bb66c3891bee60a4). 
```python
url = f"https://api.notion.com/v1/databases/{DATABASE_ID}/query"
# Получение данных из основной таблицы notion - Чистилище
try:
    res = requests.post(url, headers=headers)
    res.raise_for_status()  # Генерировать исключение, если ответ сервера не успешный
    logs("Данные из Чистилища получены")
except requests.exceptions.HTTPError as err:
    logs(f"Ошибка получении данных из Чистилища: {err}")

# Обработка данных из Чистилища Notion
data_main = res.json()
```

Заносим в переменную `main_df` список id строк в которых Исполнитель не заполнен
```python
# Список строк в которых столбец Excecutor пустая
main_df = []
for item in data_main["results"]:
    try:
        if item['properties']['Executor']['rich_text'] == []  or item['properties']['Executor']['rich_text'][0]['text']['content'] == '':
            page_id = item['id']
            chat_name = item['properties']['Chat']['title'][0]['text']['content']
            main_df.append([page_id, chat_name])
        
    except Exception as ex:
        logs(f'Ошибка при обработке данных из Чистилища: {ex}')
```

Проходим циклом по строкам и заполняем Исполнителей в соответствии названию чата.
```python
for item in main_df:
    if item[1] in owner_dict:
        executor = owner_dict[item[1]]
        logs(f"Загрузка данных для исполнителя - {executor}")
        update_data = {
            "properties": {
                "Executor": {
                    "rich_text": [{"text": {"content": executor}}]
                }
            }
        }
        
        url = f'https://api.notion.com/v1/pages/{item[0]}'

        update_response = requests.patch(url, headers=headers, json=update_data)
        logs(f'Ответ обновления данных в Чистилище: {update_response}')
```

<a name="step6"><h3>Этап №6: Выгрузка писем из Gmail</h3></a>
Логика основной выгрузки [gmail_script.py](https://github.com/DigitalStrategyMSK/DS_internal_projects/blob/pbn_skript/notion_bot/gmail_script.py)
Все вспомогательные функции, которые используются в этом скрипте можно посмотреть в [gmail_func.py](https://github.com/DigitalStrategyMSK/DS_internal_projects/blob/pbn_skript/notion_bot/gmail_func.py)
Импорт библиотек
```python
import base64
import json
from google.oauth2.credentials import Credentials
from googleapiclient.discovery import build

from gmail_func import create_page, create_email_list, get_list_email, update_executor

import requests
from datetime import datetime
import configparser
import pandas as pd
import re
from logs import logs
```

Определение токена и идентификаторов таблиц Notion
```python
config = configparser.ConfigParser()
config.read("config.ini")

# Токен аккаунта Notion
NOTION_TOKEN = config['Notion']['notion_token']
# ID базы знаний Notion
DATABASE_ID = config['Notion']['database_id_main']
# ID таблицы Почт из входящих писем
DATABASE_ID_MAIL = config['Notion']['database_id_mail']
# refresh-Токен аккаунта Notion
refresh_token = config['Notion']['refresh_token']
```

Получение доступа к Gmail Api и отправка запроса на получение списка сообщений из почты Gmail
```python
credentials_file = './client_gmail.json'
credentials = Credentials(
    token=None,
    client_id="1087085050007-he9tu21vi1pvhfhpntovnuonck0r3kaa.apps.googleusercontent.com",
    client_secret="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",
    refresh_token=refresh_token,
    token_uri='https://oauth2.googleapis.com/token'
)

# Создаем клиента Gmail API
service = build('gmail', 'v1', credentials=credentials)

results = service.users().messages().list(userId='me').execute()
messages = results.get('messages', [])
```

Получение списка email адресов, которые ранее уже присылали сообщения на почту.
```python
list_email = get_list_email(DATABASE_ID_MAIL)
```

Обрабатываем список входящих сообщений, если отправитель ранее не отправлял сообщений, то добавляем его в список новых отправителей. 
Достаем из ответа Api текст сообщения, отправителя и дату
```python
for message in messages:
    message_data = service.users().messages().get(userId='me', id=message['id']).execute()
    message_id = message['id']

    id_load_list=pd.read_csv('id_load.csv')
    # Проверка на актуальность полученного письма
    if message_id not in id_load_list['id'].tolist():
        
        def extract_message_text(message_part):
            if 'body' in message_part and 'data' in message_part['body']:
                return base64.urlsafe_b64decode(message_part['body']['data']).decode('utf-8')
            return ''

        message_parts = message_data['payload'].get('parts', [])
        message_text = ""

        message_text = extract_message_text(message_parts[0])

        # Если текст не был найден, то находим его из 'body' в 'payload'
        if not message_text:
            message_text = extract_message_text(message_data['payload'])

        subject = next(header['value'] for header in message_data['payload']['headers'] if header['name'] == 'Subject')
        sender_off = next(header['value'] for header in message_data['payload']['headers'] if header['name'] == 'From')
        date = next(header['value'] for header in message_data['payload']['headers'] if header['name'] == 'Date')
        # Преобразуем формат даты и времени
        parsed_date = datetime.strptime(date, '%a, %d %b %Y %H:%M:%S %z')
        formatted_date = parsed_date.strftime('%Y-%m-%d %H:%M:%S')

        match = re.search(r'<([^>]+)>', sender_off)
        sender = match.group(1)
```

Если отправитель новый, то грузим его в Notion таблицу [Список почты Gmail](https://www.notion.so/6d46d1ac611943689a255b39c33b77a2?v=26b4ec395fd94e8abaf8e026b91bf762)
```python
if sender not in list_email:
		data = {
			"Email": {"title": [{"text": {"content": sender}}]},
			"Owner": {"rich_text": [{"text": {"content": ""}}]}
		}
		if create_email_list(data):
			logs("Письма выгрузились в Notion")
with open('id_load.csv','a') as file:
    file.writelines(f'{message_id}\n')
```

Добавляем id сообщения в файл csv, чтобы в дальнейшем не обрабатывать это сообщение.
Формируем тело запроса `data` и загружаем сформированные данные в таблицу [Чистилище](https://www.notion.so/8a688e98892c4d9e81c4844f937a43ee?v=a44b700110884c58bb66c3891bee60a4)
```python
with open('id_load.csv','a') as file:
    file.writelines(f'{message_id}\n')
 # Оформление данных в json формат
data = {
	"Chat": {"title": [{"text": {"content": subject}}]},
	"Authors": {"rich_text": [{"text": {"content": sender}}]},
	"Published": {"date": {"start": formatted_date, "end": None}},
	"Status": {
		"type": "select",
		"select": {
			"name": "Новое"
		}
	},
	"Text": {"rich_text": [{"text": {"content": message_text}}]},
	"Executor": {"rich_text": [{"text": {"content": ""}}]}
}

if create_page(data):
	logs(f'Запись {data} добавлена!')
```

Функция обновления поля Исполнитель(по email) в основной таблице Notion - Чистилище.
```python
update_executor(DATABASE_ID_MAIL)
```
