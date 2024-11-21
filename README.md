import telebot

from telebot import types

import datetime

import requests

bot = telebot.TeleBot("7825487380:AAGqItR4LoAgYF3za-PtjkG9ei5q5Ufuw7E")


@bot.message_handler(commands=['start'])
def start(message):
    markup = create_markup()
    bot.send_message(message.chat.id, "Привет, это я", reply_markup=markup)
    bot.register_next_step_handler(message, on_click)


def on_click(message):
    if message.text == 'Узнай свое имя':
        bot.send_message(message.chat.id, f'Привет, тебя зовут {message.from_user.first_name}')
        create_markup()
    elif message.text == 'Узнай свой возраст':
        msg = bot.send_message(message.chat.id, "Пожалуйста, введите вашу дату рождения в формате YYYY-MM-DD.")
        bot.register_next_step_handler(msg, calculate_age)
    elif message.text == 'Узнай кое-что о себе':
        bio = getattr(message.from_user, 'bio', 'Нет информации о себе')
        bot.send_message(message.chat.id, f'Привет, вот что я знаю: {bio}')
        create_markup()
    elif message.text == 'Узнай какая сегодня дата':
        today_date = datetime.date.today().strftime("%Y-%m-%d")
        bot.send_message(message.chat.id, f'Привет, сегодня {today_date}')
        create_markup()
    elif message.text == 'Узнай какая сегодня погода':
        weather_data = get_weather()
        bot.send_message(message.chat.id, f'Привет, погода сегодня: {weather_data}')
        create_markup()
    cont(message)


def calculate_age(message):
    try:
        birth_date = datetime.datetime.strptime(message.text, "%Y-%m-%d")
        today = datetime.datetime.today()
        age = today.year - birth_date.year - ((today.month, today.day) < (birth_date.month, birth_date.day))
        bot.send_message(message.chat.id, f'Ваш возраст: {age} лет.')
    except ValueError:
        bot.send_message(message.chat.id, "Я не могу узнать вашу дату рождения")
    cont(message)


def get_weather(city="Moscow,RU"):
    api_key = "bb890200f9f53d3cc6d2ef71936dd7d1"
    url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}&units=metric&lang=ru"

    response = requests.get(url)
    data = response.json()

    if response.status_code == 200:
        description = data['weather'][0]['description']
        temperature = data['main']['temp']
        return f"{description}, {temperature}°C"
    else:
        return "Не удалось получить данные о погоде"


def cont(message):
    markup = create_markup()
    bot.send_message(message.chat.id, "Ты можешь узнать еще:", reply_markup=markup)
    bot.register_next_step_handler(message, on_click)


def create_markup():
    markup = types.ReplyKeyboardMarkup()
    btn1 = types.KeyboardButton('Узнай свое имя')
    btn2 = types.KeyboardButton('Узнай свой возраст')
    btn3 = types.KeyboardButton('Узнай кое-что о себе')
    btn4 = types.KeyboardButton('Узнай какая сегодня дата')
    btn5 = types.KeyboardButton('Узнай какая сегодня погода')
    markup.row(btn1, btn2, btn3, btn4, btn5)
    return markup


bot.polling(none_stop=True)
