

## Бот для генерации контента
[Ссылка на бота](https://t.me/ds_chatgpt_bot)

[Ссылка на репозиторий](https://github.com/DigitalStrategyMSK/ds_chatgpt_bot/tree/main)

Бот предназначен для генерации контента через 
API OpenAI. 

Для генерации боту нужна ссылка на гугл таблицу, где указаны запросы. 

## Содержание документации

[Импорт библиотек](#step1)

[Конфигурация параметров бота](#step2)

[Вспомогательные функции из модуля generate_lib.py](#step3)

[Основной скрипт бота main.py](#step4)



## Документация

<a name="step1"><h3>Импорт библиотек</h3></a>

```python
import asyncio
import logging
import openai
from aiogram import Bot, Dispatcher, types
from aiogram.filters.command import Command
from aiogram.filters import Command, CommandObject, CommandStart
from aiogram import F
import time
import generate_lib
import re
import traceback
import pandas as pd
```

<a name="step2"><h3>Конфигурация параметров бота</h3></a>

```python
logging.basicConfig(level=logging.INFO)
bot = Bot(token="6789983022:AAFrNfVSXswPzCmVv_5oxtWQe5ElT65K6SE")
dp = Dispatcher()
params_dict={'Версия GPT':'gpt-4-1106-preview','Имя листа в документе':'Алгоритм для GPT4 с переводом'}
dp["params"] = params_dict
```
<a name="step3"><h3>Вспомогательные функции из модуля generate_lib.py</h3></a>

В модуле generate_lib.py содержатся две функции,
которые используются в командах бота.

Первая функция get_spreadsheets_data читает
данные из гугл таблицы

```python
def get_spreadsheets_data(sheet_id: str, sheet_name: str):
    try:
        client = gspread.authorize(credentials)
        document = client.open_by_key(sheet_id)
    except:
        return 'Ошибка получения доступа к файлу, раздай доступ на редактирования для всех', False 
    
    try:
        source_sheet=document.worksheet(sheet_name)
    except:
        return 'Неверное имя листа, установи корректное имя', False 
    
    df=get_as_dataframe(source_sheet)
    df.dropna(how='all',inplace=True)

    return df, source_sheet
```
Вторая функция open_ai_request отправляет запросы
в API OpenAI для генерации текста
```python
def open_ai_request(messages,gpt_version,try_count=0):
    response=openai.ChatCompletion.create(
            model=gpt_version,
            messages=messages
            )
    
    message_to_user=''
    for choice in response.choices:
        message_to_user+=choice['message']['content']+'\n'
    return message_to_user, True
```
Также в этом модуле содержатся необходимые 
параметры для подключения к гугл таблицам и 
к openai

<a name="step4"><h3>Основной скрипт бота main.py</h3></a>

Бот написан на асинхронной библиотеке aiogram

В него заложены следующие обработчики команд/сообщений:

1. Обработчик команды /start

При вызове команды /start на клавиатуре 
отобразятся несколько кнопок ответа боту:
- Выбрать версию GPT
- Показать параметры
- Изменить имя листа в документе
- Начать генерацию
- Остановить генерацию (это кнопка никак не обрабатывается)

```python
@dp.message(Command("start"))
async def cmd_start(message: types.Message):
    generate_lib.change_stopped(dp)
    keyboard=return_keyboard(['Выбрать версию GPT','Показать параметры','Изменить имя листа в документе','Начать генерацию','Остановить генерацию'])
    await message.answer("Вот, что ты можешь со мной сделать",reply_markup=keyboard)
```
2. Обработчик сообщения "Выбрать версию GPT"

При ответе на это сообщение бот выведет подсказку,
в которой указаны, какие версии есть на выбор.
```python
@dp.message(F.text.lower() == "выбрать версию gpt")
async def choose_gpt(message: types.Message):
    keyboard=return_keyboard(['GPT-4','GPT-4 Turbo', 'GPT-3.5 Turbo'])
    await message.reply("Какая версия?",reply_markup=keyboard)
```
При нажатии на ту или иную версию бот запишет
в параметры выбранную версию (для каждой версии есть обработчик, который перезаписывает параметр), она далее будет передана в функцию генерации контента. 

3. Обработчик сообщения "Показать параметры"

Данный обработчик выводит список текущих заданных параметров для генерации текста, а именно:
- Версия GPT
- Имя листа в документе (данный параметр отвечает за название листа в гугл таблице для того, чтобы можно было его прочитать)
```python
@dp.message(F.text.lower() == "показать параметры")
async def show_params(message: types.Message, params: dict):
    keyboard=return_keyboard(['Выбрать версию GPT','Показать параметры','Изменить имя листа в документе','Начать генерацию','Остановить генерацию'])
    await message.answer('\n'.join([f'{key}: {value}' for key,value in params.items()]),reply_markup=keyboard)
```
4. Обработчик сообщения "Изменить имя листа в документе"

Обработчик отвечает на сообщение с текстом:
"Напишите в формате 'Имя листа: Лист1'".

```python
@dp.message(F.text.lower() == "изменить имя листа в документе")
async def change_sheet_name_message(message: types.Message):
    await message.reply('Напишите в формате "Имя листа: Лист1"')
```
То есть это подсказка, которая указывает на то,
как написать сообщение боту, чтобы он изменил текущий параметр имя листа. При написании сообщения Имя листа: Лист1 будет задействован другой обработчик, который изменит текущий параметр имени листа в гугл таблице. 
```python
@dp.message(F.text.lower().startswith('имя листа: '))
async def change_sheet_name_function(message: types.Message, params: dict):
    params['Имя листа в документе']=message.text.replace('Имя листа: ','')
    keyboard=return_keyboard(['Выбрать версию GPT','Показать параметры','Изменить имя листа в документе','Начать генерацию','Остановить генерацию'])
    await message.answer("Вот, что ты можешь со мной сделать",reply_markup=keyboard)
```
5. Обработчик "Начать генерацию"

Этот обработчик отвтеит списком текущих параметров, а также попросит прислать ссылку на гугл таблицы, чтобы начать генерацию контента.
```python
@dp.message(F.text.lower() == "начать генерацию")
async def start_generation_question(message: types.Message, is_stopped: list, params: dict):
    await message.answer('Выставлены такие параметры:\n\n'+'\n'.join([f'{key}: {value}' for key,value in params.items()]))
    await message.answer('Пришли ссылку на файл Spreadsheets если хочешь начать генерацию с текущими параметрами')
```
Когда боту отправляется ссылка на гугл таблицу, запускается еще один обработчик, который парсит ссылку на гугл таблицы и начинает обрабатывать файл.
```python
@dp.message(F.text.lower().startswith("https://docs.google.com/spreadsheets/"))
async def start_generation_function(message: types.Message, params: dict):
    match = re.search('/spreadsheets/d/([\w-]+)', message.text)
    if match:
        sheet_id = match.group(1)
    else:
        await message.answer('Некорректная ссылка, повтори попытку')
    data=generate_lib.get_spreadsheets_data(sheet_id, params['Имя листа в документе'])
    if not data[1]:
        await message.answer(data[0])
    else:
        df,source_sheet=data
    df.loc[:,'gpt_version']=params['Версия GPT']
    try:
        df.apply(get_text, axis=1, args=(source_sheet,))
    except openai.error.RateLimitError:
        # Обработка ошибки RateLimitError
        await message.answer('Биллинговые лимиты закончились. Пожалуйста, проверьте ваш аккаунт и увеличьте лимиты.')
        await bot.send_message(512005589, 'Биллинговые лимиты закончились')
    except Exception:
        error_message=traceback.format_exc()
        await bot.send_message(663569655,'Ошибка генерации текстов ПБН:\n'+error_message)
        await bot.send_message(512005589,'Ошибка генерации текстов ПБН:\n'+error_message)
        await message.answer('Ошибка генерации текста')
    await message.answer('Генерация закончена')
```
Внутри обработчика также сидит функция get_text(), которая парсит строки таблицы, отправляет запросы в openai через функцию open_ai_request и добавляет полученный результат к текущей гугл таблице через функцию write_column().

После завершение генерации бот выводит сообщение 
"Генерация закончена". 

Ниже код вспомогательных функций:
- get_text
```python
def get_text(row,source_sheet):
    # если строка готового текста заполнена, то не обрабатываем ее
    if pd.notnull(row['Готовый текст']):
        return row

    messages=[
            {"role": "system", "content": "You are a experienced copywriter."},
    ]

    # обрабатываем каждый столбец датафрейма
    for name in row.index:
        final_text=False
        # обрабатываем колонки, которые содержат слово запрос в названии
        if 'запрос' in name.lower():
            # если в названии содержится "(", то забираем параметр внутри скобок и вставляем вместо ключевого слова в запросе
            if '(' in name:
                var=re.findall('\((.+)\)',name)[0]
                messages.append({"role": "user", "content": str(row[name]).replace(f'[{var}]',str(row[var]))})
            else:
                messages.append({"role": "user", "content": row[name]})
            # отправляем запрос в openai
            message_to_user,status=generate_lib.open_ai_request(messages,row['gpt_version'])
            # если -> содержится в запросе, то записываем вв результат в колонку с названием после ->
            if '->' in name:
                var=re.findall('\-\>(.+)$',name)[0]
                write_column(source_sheet,message_to_user,list(row.index).index(var),row.name+2)
                # вставляем текст с переводом
                if row['Переводим?'].lower()=='да':
                    write_column(source_sheet,generate_lib.translate(row['Язык оригинала'],row['Язык перевода'], message_to_user),list(row.index).index(var+'[translated]'),row.name+2)
                if 'готовый текст' in name.lower():
                    final_text=message_to_user
                if final_text:
                    messages=[
                        {"role": "system", "content": final_text},
                        ]
                else:
                    messages.append({"role": "system", "content": message_to_user})
            else:
                messages.append({"role": "system", "content": message_to_user})
```
- write_column
```python
def write_column(source_sheet, value: str, df_column_index: int, df_row_index: int):
    body=[value]
    df_column_name=generate_lib.col_names[df_column_index]
    source_sheet.append_row(body, table_range=f"{df_column_name}{df_row_index}:{df_column_name}{df_row_index}")
```














