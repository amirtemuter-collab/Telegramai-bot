import os
import asyncio
from telegram import Update
from telegram.ext import (
    Application,
    CommandHandler,
    MessageHandler,
    ContextTypes,
    filters,
)
from groq import Groq

TELEGRAM_TOKEN = os.environ["TELEGRAM_TOKEN"]
GROQ_API_KEY = os.environ["GROQ_API_KEY"]

client = Groq(api_key=GROQ_API_KEY)

# حافظه موقت مکالمه برای هر کاربر
user_history = {}

SYSTEM_PROMPT = """
تو یک دستیار هوش مصنوعی شخصی هستی.
به فارسی روان، طبیعی و دقیق پاسخ بده.
اگر کاربر سؤال درسی پرسید، مرحله‌به‌مرحله و قابل فهم توضیح بده.
اگر مطمئن نیستی، حدس نزن و صادقانه بگو.
لحن دوستانه و طبیعی داشته باش، اما اطلاعات غلط نده.
"""

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "سلام 👋\nمن دستیار هوش مصنوعی شخصی تو هستم.\nهر چیزی می‌خوای بپرس."
    )


async def chat(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    text = update.message.text

    if user_id not in user_history:
        user_history[user_id] = []

    history = user_history[user_id]

    messages = [
        {"role": "system", "content": SYSTEM_PROMPT}
    ]

    messages.extend(history[-10:])
    messages.append({"role": "user", "content": text})

    try:
        completion = await asyncio.to_thread(
            client.chat.completions.create,
            model="openai/gpt-oss-120b",
            messages=messages,
            temperature=0.7,
            max_tokens=2048,
        )

        answer = completion.choices[0].message.content

        history.append({"role": "user", "content": text})
        history.append({"role": "assistant", "content": answer})

        await update.message.reply_text(answer)

    except Exception as e:
        print("ERROR:", e)
        await update.message.reply_text(
            "فعلاً نتونستم جواب بدم 😕\nچند لحظه دیگه دوباره امتحان کن."
        )


def main():
    app = Application.builder().token(TELEGRAM_TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(
        MessageHandler(filters.TEXT & ~filters.COMMAND, chat)
    )

    print("Bot is running...")
    app.run_polling()


if __name__ == "__main__":
    main()  
