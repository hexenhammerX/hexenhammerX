#!/usr/bin/python3

import requests
import json

from aiogram import Bot, types
from aiogram.dispatcher import Dispatcher
from aiogram.utils import executor
from aiogram.types import ReplyKeyboardRemove, \
    ReplyKeyboardMarkup, KeyboardButton, \
    InlineKeyboardMarkup, InlineKeyboardButton

from aiogram.types import BotCommand
from aiogram.utils.helper import Helper, HelperMode, ListItem
from aiogram.contrib.fsm_storage.memory import MemoryStorage
from aiogram.contrib.middlewares.logging import LoggingMiddleware
from aiogram.dispatcher import FSMContext
from aiogram.dispatcher.filters.state import State, StatesGroup
from aiogram.utils.callback_data import CallbackData
from aiogram.dispatcher.filters import Text
from aiogram.utils.markdown import hlink

import logging

######## Aiogram системные переменные ########

logging.basicConfig(level=logging.INFO)

bot = Bot(token='5432519427:AAEJlcq3L7r-qI68mCz21SmLCUdifsCHSi0')

API_KEY = 'bb3dd426f9b9c2a97a32ff73493d86bc15697bfb'

dp = Dispatcher(bot, storage=MemoryStorage())
dp.middleware.setup(LoggingMiddleware())
data = {}


async def set_bot_commands(bot: Bot):
    bot_commands = [
        BotCommand(command="/help", description="Get info about me"),
        BotCommand(command="/qna", description="set bot for a QnA task"),
        BotCommand(command="/chat", description="set bot for free chat")
    ]
    # await bot.set_my_commands(bot_commands)
    # for command in commands_list:
    await bot.set_my_commands(bot_commands)


# Обработка команды /start
@dp.message_handler(commands='start')
async def cmd_start(message: types.Message):
    await message.answer('Выберите действие')


# Обработка запросов к Redmine пользователей прошедших авторизацию
@dp.message_handler()
async def filter_options(message: types.Message, state: FSMContext):
    res1 = requests.get('https://api-fns.ru/api/search?q=' + message.text + '&key=' + API_KEY)
    print(res1)
    json_res1 = res1.json()
    print(json_res1)

    res2 = requests.get(
        'https://api-fns.ru/api/egr?req=' + json_res1['items'][0]['ЮЛ']['ИНН'] + '&key=' + API_KEY)
    json_res2 = res2.json()
    print(json_res2)

    res3 = requests.get(
        'https://api-fns.ru/api/multinfo?req=' + json_res1['items'][0]['ЮЛ']['ИНН'] + ',' + json_res1['items'][0]['ЮЛ'][
            'ОГРН'] + '&key=' + API_KEY)
    json_res3 = res3.json()
    print(json_res3)

    message_info = (json_res1['items'][0]['ЮЛ']['ИНН']
                    + '\nОГРН: ' + json_res1['items'][0]['ЮЛ']['ОГРН']
                    + '\nАдресПолн: ' + json_res1['items'][0]['ЮЛ']['АдресПолн']
                    + '\nДатаРег: ' + json_res2['items'][0]['ЮЛ']['ДатаРег']
                    + '\nУставный' + ' ' + 'капитал: ' + json_res2['items'][0]['ЮЛ']['Капитал']['СумКап']
                    + '\nКод: ' + json_res2['items'][0]['ЮЛ']['ОснВидДеят'][
                        'Код'] + ' ' + 'Вид' + ' ' + 'деятельности: ' + json_res2['items'][0]['ЮЛ']['ОснВидДеят'][
                        'Текст']
                    + '\nРегистратор: ' + json_res2['items'][0]['ЮЛ']['НО']['Рег']
                    + '\nГенеральный директор: ' + json_res2['items'][0]['ЮЛ']['Руководитель']['ФИОПолн']
                    + '\nИНН: ' + json_res2['items'][0]['ЮЛ']['Руководитель']['ИННФЛ']
                    + '\nУчредители: ' + json_res2['items'][0]['ЮЛ']['Учредители'][0]['УчрФЛ']['ФИОПолн']
                    + '\nВыручка от продажи за 2021: ' + json_res3['items'][0]['ЮЛ']['Финансы']['Выручка']
                    + '\nСреднесписочная численность: ' + json_res2['items'][0]['ЮЛ']['ОткрСведения']['КолРаб'])
                    #+ '\nЧистая прибыль за 2021: ' + json_res2['items'][0]['ЮЛ']['ОткрСведения'][0]['Налоги'][0]['СумДоход'] - json_res2['items'][0]['ЮЛ']['ОткрСведения']['СумРасход'])

    #Исполнительное производство:


    await message.answer(message_info)


if __name__ == '__main__':
    # scheduler.start()
    executor.start_polling(dp, skip_updates=True)
