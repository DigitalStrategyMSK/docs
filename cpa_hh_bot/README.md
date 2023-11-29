[Ссылка на репозиторий проекта](https://github.com/DigitalStrategyMSK/cpa_hh_bot)

Скрипт лежит на сервере 176.112.209.190 и запускается раз в 10 минут


## Этап №1: Импорт библиотек

```python
import httplib2
from oauth2client.service_account import ServiceAccountCredentials
import pandas as pd
from google.oauth2 import service_account
from pprint import pprint
import numpy as np
from clickhouse_driver import Client
from googleapiclient.errors import HttpError
import os.path
import base64
from bs4 import BeautifulSoup
import re
from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build
import datetime
import traceback
import telebot
import sys
```

## Этап №2: Объявление констант и функции отправки сообщений об ошибках

```python
SCOPES = ['https://www.googleapis.com/auth/gmail.readonly','https://www.googleapis.com/auth/gmail.modify']

error_bot = telebot.TeleBot(
    '5883566262:AAE6QhNiZgTEiIMhBwmgx_HKYuXglQqmdBE', parse_mode=None)

bot = telebot.TeleBot('6475373291:AAF4HSg4zcb1foXlHBY3yRcPuF-3HXQQqdM', parse_mode='html')

def set_error_message(text,error_message):
    error_bot.send_message(663569655, f'Произошла ошибка боте отправки cpa.hh.ru:\n{text}\n{error_message}')
    sys.exit()
```

## Этап №3: Объявление функции получения сообщений

### Называем функцию

```python
def get_email_messages():
```

### Получаем данные для доступа к API Gmail

Считываем файл token.json если он существует и записываем в переменную creds
```python
    creds = None
    if os.path.exists('token.json'):
        creds = Credentials.from_authorized_user_file('token.json', SCOPES)
```

Если переменная creds равна False или невалидная для авторизации, то есть 2 пути:
1. Обновить creds с помощью refresh_token в случае если данные авторизации истекли и refresh_token есть 
```python
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
```

2. Считать данные для авторизации из файла creds.json
```python
        else:
            flow = InstalledAppFlow.from_client_secrets_file(
                'creds.json', SCOPES)
            creds = flow.run_local_server(port=0)
```

Записываем в файл token.json новые данные для авторизации
```python
        with open('token.json', 'w') as token:
            token.write(creds.to_json())
```

### Подключаемся к сервису Gmail с помощью данные авторизации и собираем письма с пометкой *непрочитанное*

```python
    try:
        service = build('gmail', 'v1', credentials=creds)
        results = service.users().messages().list(
            userId='cpa.hh.tg@gmail.com',
            labelIds=['INBOX'],
            q="is:unread"
        ).execute()
        messages = results.get('messages',[])
```

### Проходимся по списку сообщений и собираем текст, автора и заголовок сообщения и делаем эти письма *прочитанными*

```python
        if messages:
            for message in messages:
                msg = service.users().messages().get(userId='cpa.hh.tg@gmail.com', id=message['id']).execute()
                byte_code = base64.urlsafe_b64decode(msg['payload']['body']['data'])
                text = byte_code.decode("utf-8")
                for header in msg['payload']['headers']:
                    if header['name']=='Return-Path':
                        email_from=header['value']
                    elif header['name']=='Subject':
                        title=header['value']
                service.users().messages().modify(userId='me', id=msg['id'], body={'removeLabelIds': ['UNREAD']}).execute()
                result.append([msg['id'],title,text])
            return result
        return messages
```

Если на этапе сбора сообщений произошла ошибка, то уведомляем о ней
```python
    except Exception:
        error_message=traceback.format_exc()
        set_error_message('Ошибка при сборе сообщений',error_message)
        return 0
```

## Этап №4: Обработка текста сообщения

### Получаем данные новых сообщений в цикле и парсим их html разметку
```python
try:
    messages_list=get_email_messages()

    for row in messages_list[:]:
        try:
            #парсим текст как html разметку
            soup = BeautifulSoup(row[2], "html.parser")
```

### Форматируем текст сообщений
```python
            #заменяем некоторые сочетания тегов на пропуск строки
            text_to_tg=str(soup).replace('<br>','\n')
            text_to_tg=text_to_tg.replace('<br/>','\n')
            text_to_tg=text_to_tg.replace('<p><li>','\n')
            text_to_tg=text_to_tg.replace('<p>','\n')
            
            #оставляем в разметке только те теги, которые могут использоваться в Telegram
            allowed_tags = ["b", "i", "a", "code", "pre"]
            pattern = r"<(?!\/?(?:{})\b)[^>]+>".format("|".join(allowed_tags))
            result = re.sub(pattern, "", text_to_tg)
            
            #заменяем ключевой символ ` на абзац
            result=result.replace('`','\n')
            
            #добавляем заголовок из письма в сообщение
            result=('<b>'+row[1]+'</b>'+'\n\n'+re.sub('^\n+', "", result))
            
            #добавляем абзац перед тегом
            substring=re.search('(#новости|#запуск|#возобновление|#остановка|#изменение)', result)
            result=re.sub(f'{substring.group(1)}', f"\n{substring.group(1)}", result)

            #обрезаем сообщение до ключевого тега из списка
            result=result[:result.index(substring.group(1))+len(substring.group(1))]
```

### Формируем и отправляем сообщение в [Telegram-канал](https://t.me/cpa_hh_ru)

Для каждого из тегов(которых всего 5) внутри репозитория есть соответствующая картинка для отправки сообщения в Telegram
```python
            #отправляем сообщение с картинкой соответствующей каждому из тегов
            tags={'#запуск':'start','#возобновление':'restart','#изменение':'change','#новости':'news','#остановка':'stop'}
            for tag in tags.keys():
                if tag in result:
                    bot.send_photo(-1001927773287, photo=open(f'{tags[tag]}.jpg', 'rb'), caption=result, parse_mode='html')
```

### В случае возникновения ошибки на этапе обработки и отправки сообщения шлем сообщение об ошибке
```python
        except Exception as e:
            set_error_message('Ошибка при выполнении отправки сообщения',e)
            continue
except Exception:
    error_message=traceback.format_exc()
    set_error_message('Ошибка при выполнении парсинга и отправки сообщений',error_message)
```
