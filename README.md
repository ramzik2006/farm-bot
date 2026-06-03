import telebot
from telebot.types import InlineKeyboardMarkup, InlineKeyboardButton

TOKEN = "8835722761:AAEqL_a8IGDIB0EtCyGDj6tQnsCXZDRMHIO"

bot = telebot.TeleBot(TOKEN)

players = {}

def get_keyboard():
    keyboard = InlineKeyboardMarkup()
    keyboard.row(InlineKeyboardButton("🌾 Покормить курицу", callback_data="feed"))
    keyboard.row(InlineKeyboardButton("📊 Статистика", callback_data="stats"))
    keyboard.row(InlineKeyboardButton("✨ Улучшение (50 зелий)", callback_data="upgrade"))
    return keyboard

@bot.message_handler(commands=['start'])
def start(message):
    user_id = message.from_user.id
    if user_id not in players:
        players[user_id] = {"coins": 0, "bonus": 1}
    bot.send_message(message.chat.id, "🐔 Добро пожаловать в Ферму!\nКорми курицу и получай зелья!", reply_markup=get_keyboard())

@bot.callback_query_handler(func=lambda call: True)
def callback(call):
    user_id = call.from_user.id
    if user_id not in players:
        players[user_id] = {"coins": 0, "bonus": 1}
    
    if call.data == "feed":
        players[user_id]["coins"] += 1 * players[user_id]["bonus"]
        bot.answer_callback_query(call.id, f"+{1 * players[user_id]['bonus']} зелья!")
        bot.edit_message_text(f"🐔 Курица накормлена!\n🧪 Зелий: {players[user_id]['coins']}", call.message.chat.id, call.message.message_id, reply_markup=get_keyboard())
    
    elif call.data == "stats":
        coins = players[user_id]["coins"]
        bonus = players[user_id]["bonus"]
        bot.send_message(call.message.chat.id, f"📊 Твоя ферма:\n🧪 Зелий: {coins}\n⚡ Множитель: x{bonus}")
    
    elif call.data == "upgrade":
        if players[user_id]["coins"] >= 50:
            players[user_id]["coins"] -= 50
            players[user_id]["bonus"] += 1
            bot.answer_callback_query(call.id, "✨ Улучшение куплено!")
            bot.send_message(call.message.chat.id, f"🎉 Теперь x{players[user_id]['bonus']} зелий за кормёжку!")
        else:
            bot.answer_callback_query(call.id, "❌ Нужно 50 зелий!")

print("🚀 Бот запущен!")
bot.infinity_polling()
