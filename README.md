import requests

from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import (
    Application,
    CommandHandler,
    CallbackQueryHandler,
    ContextTypes
)


# ==============================
# НАСТРОЙКИ
# ==============================

TELEGRAM_TOKEN = "8947008258:AAH5xdjnOSZQRR2Vrcegr3gJVxYz-gEYpn4"
WEATHER_API_KEY = "0b5f4a9e11e950962ad2e343a8075ab3"


# Координаты городов Казахстана
CITIES = {
    "almaty": {
        "name": "Алматы",
        "lat": 43.2389,
        "lon": 76.8897
    },

    "astana": {
        "name": "Астана",
        "lat": 51.1694,
        "lon": 71.4491
    },

    "shymkent": {
        "name": "Шымкент",
        "lat": 42.3417,
        "lon": 69.5901
    }
}


# ==============================
# КОМАНДА /start
# ==============================

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):

    keyboard = [
        [
            InlineKeyboardButton(
                "🏔 Алматы",
                callback_data="almaty"
            )
        ],
        [
            InlineKeyboardButton(
                "🏛 Астана",
                callback_data="astana"
            )
        ],
        [
            InlineKeyboardButton(
                "🌆 Шымкент",
                callback_data="shymkent"
            )
        ]
    ]

    keyboard_markup = InlineKeyboardMarkup(keyboard)

    await update.message.reply_text(
        "🌤 Добро пожаловать!\n\n"
        "Выберите город, чтобы узнать текущую погоду:",
        reply_markup=keyboard_markup
    )


# ==============================
# ПОЛУЧЕНИЕ ПОГОДЫ
# ==============================

def get_weather(city):

    data = CITIES[city]

    url = "https://api.openweathermap.org/data/2.5/weather"

    params = {
        "lat": data["lat"],
        "lon": data["lon"],
        "appid": WEATHER_API_KEY,
        "units": "metric",
        "lang": "ru"
    }

    response = requests.get(url, params=params)

    if response.status_code != 200:
        return None

    weather = response.json()

    temperature = weather["main"]["temp"]
    feels_like = weather["main"]["feels_like"]
    humidity = weather["main"]["humidity"]
    pressure = weather["main"]["pressure"]
    wind = weather["wind"]["speed"]

    description = weather["weather"][0]["description"]

    return {
        "city": data["name"],
        "temperature": temperature,
        "feels_like": feels_like,
        "humidity": humidity,
        "pressure": pressure,
        "wind": wind,
        "description": description
    }


# ==============================
# ОБРАБОТКА КНОПОК
# ==============================

async def city_button(update: Update, context: ContextTypes.DEFAULT_TYPE):

    query = update.callback_query

    # Подтверждаем нажатие кнопки
    await query.answer()

    city = query.data

    weather = get_weather(city)

    if weather is None:

        await query.edit_message_text(
            "❌ Не удалось получить данные о погоде."
        )

        return


    text = (
        f"🌤 <b>Погода: {weather['city']}</b>\n\n"

        f"🌡 Температура: "
        f"<b>{weather['temperature']:.1f} °C</b>\n"

        f"🤔 Ощущается как: "
        f"<b>{weather['feels_like']:.1f} °C</b>\n\n"

        f"☁️ Состояние: "
        f"{weather['description'].capitalize()}\n"

        f"💧 Влажность: "
        f"{weather['humidity']}%\n"

        f"💨 Ветер: "
        f"{weather['wind']} м/с\n"

        f"🧭 Давление: "
        f"{weather['pressure']} гПа"
    )


    # Кнопка для повторного выбора города
    keyboard = [
        [
            InlineKeyboardButton(
                "🏙 Выбрать другой город",
                callback_data="back"
            )
        ]
    ]

    keyboard_markup = InlineKeyboardMarkup(keyboard)


    await query.edit_message_text(
        text=text,
        parse_mode="HTML",
        reply_markup=keyboard_markup
    )


# ==============================
# КНОПКА "НАЗАД"
# ==============================

async def back_button(update: Update, context: ContextTypes.DEFAULT_TYPE):

    query = update.callback_query

    await query.answer()

    keyboard = [
        [
            InlineKeyboardButton(
                "🏔 Алматы",
                callback_data="almaty"
            )
        ],
        [
            InlineKeyboardButton(
                "🏛 Астана",
                callback_data="astana"
            )
        ],
        [
            InlineKeyboardButton(
                "🌆 Шымкент",
                callback_data="shymkent"
            )
        ]
    ]

    keyboard_markup = InlineKeyboardMarkup(keyboard)

    await query.edit_message_text(
        "🌤 Выберите город:",
        reply_markup=keyboard_markup
    )


# ==============================
# ЗАПУСК БОТА
# ==============================

def main():

    application = (
        Application.builder()
        .token(TELEGRAM_TOKEN)
        .build()
    )

    # /start
    application.add_handler(
        CommandHandler("start", start)
    )

    # Кнопки городов
    application.add_handler(
        CallbackQueryHandler(
            city_button,
            pattern="^(almaty|astana|shymkent)$"
        )
    )

    # Кнопка назад
    application.add_handler(
        CallbackQueryHandler(
            back_button,
            pattern="^back$"
        )
    )

    print("Бот запущен...")

    application.run_polling()


if __name__ == "__main__":
    main()


