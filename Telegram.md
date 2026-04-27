import os
import requests
from telegram import Update
from telegram.ext import ApplicationBuilder, MessageHandler, filters, ContextTypes

BOT_TOKEN = os.getenv("8693511131:AAHAPf19ZAya8iFJqOboJQPXrQL_nebANuc")
GROQ_API_KEY = os.getenv("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")

async def handle_audio(update: Update, context: ContextTypes.DEFAULT_TYPE):
    file = await update.message.audio.get_file()
    file_path = "audio.mp3"
    await file.download_to_drive(file_path)

    url = "https://api.groq.com/openai/v1/audio/transcriptions"

    headers = {
        "Authorization": f"Bearer {GROQ_API_KEY}"
    }

    files = {
        "file": open(file_path, "rb"),
        "model": (None, "whisper-large-v3"),
        "response_format": (None, "srt")
    }

    response = requests.post(url, headers=headers, files=files)

    with open("output.srt", "w", encoding="utf-8") as f:
        f.write(response.text)

    await update.message.reply_document(document=open("output.srt", "rb"))

app = ApplicationBuilder().token(BOT_TOKEN).build()
app.add_handler(MessageHandler(filters.AUDIO, handle_audio))

app.run_polling()
