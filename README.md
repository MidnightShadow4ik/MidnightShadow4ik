import telebot
import sqlite3

# Создание экземпляра бота с токеном
bot = telebot.TeleBot("6575540038:AAHdSu5F2r6lR44Xtbf8-6hmco84lrH9t2I")

# Подключение к базе данных SQLite
conn = sqlite3.connect('job_applications.db')
cursor = conn.cursor()

# Создание таблицы в базе данных, если она еще не существует
cursor.execute('''CREATE TABLE IF NOT EXISTS applicants
                  (id INTEGER PRIMARY KEY AUTOINCREMENT,
                  name TEXT,
                  age INTEGER,
                  position TEXT,
                  experience TEXT)''')

# Обработчик команды /start
@bot.message_handler(commands=['start'])
def send_welcome(message):
    bot.reply_to(message, "Привет! Я бот, который принимает заявки на работу. Для начала работы, отправь мне свою заявку.")

# Обработчик сообщений с информацией о соискателе
@bot.message_handler(func=lambda message: True)
def handle_job_application(message):
    job_application = message.text.split(',')  # Предполагаем формат: "Имя, Возраст, Позиция, Опыт работы"
    
    # Проверка корректности формата данных
    if len(job_application) != 4:
        bot.reply_to(message, "Неверный формат данных. Пожалуйста, попробуйте еще раз.")
        return
    
    # Разделение данных по соответствующим полям
    name = job_application[0].strip()
    age = int(job_application[1].strip())
    position = job_application[2].strip()
    experience = job_application[3].strip()
    
    # Добавление записи с заявкой в базу данных
    cursor.execute("INSERT INTO applicants (name, age, position, experience) VALUES (?, ?, ?, ?)",
                   (name, age, position, experience))
    conn.commit()
    
    bot.reply_to(message, "Ваша заявка успешно принята. Мы свяжемся с вами в случае необходимости.")

# Обработчик команды /applicants
@bot.message_handler(commands=['applicants'])
def list_applicants(message):
    cursor.execute("SELECT * FROM applicants")
    all_applicants = cursor.fetchall()
    
    if all_applicants:
        response = "Список всех соискателей на работу:\n"
        for applicant in all_applicants:
            response += f"ID: {applicant[0]}, Имя: {applicant[1]}, Возраст: {applicant[2]}, Позиция: {applicant[3]}, Опыт работы: {applicant[4]}\n"
    else:
        response = "Нет заявок на работу."
    
    bot.reply_to(message, response)

# Запуск бота
bot.polling()
.............................
В этом примере бот принимает заявки на работу в формате "Имя, Возраст, Позиция, Опыт работы". Он сохраняет эти данные в таблицу applicants базы данных SQLite. Команда /applicants используется для получения списка всех заявок на работу.

