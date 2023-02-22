import telebot
import sqlite3
from telebot import types

# Создаем соединение с базой данных
conn = sqlite3.connect('database.db')
cursor = conn.cursor()

# Создаем бота
bot = telebot.TeleBot('YOUR_TOKEN')

# Определяем обработчик для команды /start
@bot.message_handler(commands=['start'])
def send_welcome(message):
    chat_id = message.chat.id
    # Создаем клавиатуру для выбора региона
    regions_keyboard = types.ReplyKeyboardMarkup(resize_keyboard=True)
    regions_keyboard.add('Москва', 'Санкт-Петербург')
    bot.send_message(chat_id, 'Выберите регион:', reply_markup=regions_keyboard)

# Определяем обработчик для сообщения с выбранным регионом
@bot.message_handler(func=lambda message: message.text in ['Москва', 'Санкт-Петербург'])
def handle_region(message):
    chat_id = message.chat.id
    region = message.text
    # Получаем список городов для выбранного региона из базы данных
    cursor.execute(f"SELECT DISTINCT city FROM files WHERE region='{region}'
@bot.message_handler(func=lambda message: message.text in ['Москва', 'Санкт-Петербург'])
def handle_region(message):
    chat_id = message.chat.id
    region = message.text
    # Получаем список городов для выбранного региона из базы данных
    cursor.execute(f"SELECT DISTINCT city FROM files WHERE region='{region}'")
    cities = [row[0] for row in cursor.fetchall()]
    # Создаем клавиатуру для выбора города
    cities_keyboard = types.ReplyKeyboardMarkup(resize_keyboard=True)
    for city in cities:
        cities_keyboard.add(city)
    cities_keyboard.add('Главное меню')
    bot.send_message(chat_id, 'Выберите город:', reply_markup=cities_keyboard)

# Определяем обработчик для сообщения с выбранным городом
@bot.message_handler(func=lambda message: True)
def handle_city(message):
    chat_id = message.chat.id
    city = message.text
    # Если пользователь выбрал "Главное меню", показываем клавиатуру с регионами
    if city == 'Главное меню':
        regions_keyboard = types.ReplyKeyboardMarkup(resize_keyboard=True)
        regions_keyboard.add('Москва', 'Санкт-Петербург')
        bot.send_message(chat_id, 'Выберите регион:', reply_markup=regions_keyboard)
        return
    # Получаем список файлов для выбранного города из базы данных
    cursor.execute(f"SELECT * FROM files WHERE city='{city}'")
    files = cursor.fetchall()
    # Создаем клавиатуру для выбора файла
    files_keyboard = types.InlineKeyboardMarkup()
    for file in files:
        file_id = file[0]
        file_name = file[1]
        file_description = file[2]
        file_price = file[3]
        # Создаем кнопку для каждого файла
        file_button = types.InlineKeyboardButton(text=f'{file_name} ({file_price} руб.)', callback_data=f'buy_{file_id}')
        files_keyboard.add(file_button)
    files_keyboard.add(types.InlineKeyboardButton(text='Главное меню', callback_data='back_to_menu'))
    bot.send_message(chat_id, f'Выберите файл для покупки в городе {city}:', reply_markup=files_keyboard)

# Определяем обработчик для инлайн-кнопки "Купить"
@bot.callback_query_handler(func=lambda call: call.data.startswith('buy_'))
def handle_buy(call):
    chat_id = call.message.chat.id
    file_id = call.data[4:]
    # Получаем информацию о выбранном файле из базы данных
    cursor.execute(f"SELECT * FROM files WHERE id={file_id}")
    file_info = cursor.fetchone()
    file_name = file_info[1]
    file_price = file_info[3]
    # Отправляем сообщение пользователю с информацией о заказе
    bot.send_message(chat_id, f'Вы выбрали файл "{file_name}" за {file_price} руб.\n'
                              f'Пожалуйста, выберите способ оплаты и свяжитесь с нами для
# Определяем обработчик для сообщения с информацией о заказе и опциями оплаты
@bot.message_handler(func=lambda message: True)
def handle_order(message):
    chat_id = message.chat.id
    # Отправляем сообщение пользователю с информацией о заказе и опциями оплаты
    bot.send_message(chat_id, 'Спасибо за заказ! Пожалуйста, выберите способ оплаты:\n'
                              '1. Оплата картой на сайте\n'
                              '2. Оплата наличными при получении\n'
                              '3. Оплата по банковскому переводу\n'
                              'После оплаты свяжитесь с нами для получения файла.')

# Определяем обработчик для инлайн-кнопки "Главное меню"
@bot.callback_query_handler(func=lambda call: call.data == 'back_to_menu')
def handle_back_to_menu(call):
    chat_id = call.message.chat.id
    regions_keyboard = types.ReplyKeyboardMarkup(resize_keyboard=True)
    regions_keyboard.add('Москва', 'Санкт-Петербург')
    bot.send_message(chat_id, 'Выберите регион:', reply_markup=regions_keyboard)

# Запускаем бота
bot.polling()
