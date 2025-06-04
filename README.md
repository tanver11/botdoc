from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler, ContextTypes

# Dictionary of questions with answers or PDF URLs
qa = {
    "Who am I?": """I am a Telegram bot. I will assist you with all semester questions and other documents related to the Statistics Department of Comilla University. I was created by Tanver Bhuiyan (Stat-15).
""",
    "Release Date?": "02-Jun-2025."
}

# Dictionary of PDFs (replace these URLs with your Google Drive direct download links)
pdf_files = {
    "Curriculum": "https://drive.google.com/uc?export=download&id=1KMYI0xEl5jjS4cfQJxDXYobrHIWSFeIs",
    "13 Batch 6th Semester": "https://drive.google.com/uc?export=download&id=1LXQjnpClk7mLQ-O-BcA0-OP5q9hmw-Y-",
    "14 Batch 6th Semester": "https://drive.google.com/uc?export=download&id=1ZV8f65tomH9MPK87cnbJnALwMPUoaE4j"
}


async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = [
        [InlineKeyboardButton("Ask Questions", callback_data="ask_questions")],
        [InlineKeyboardButton("Get Semester PDFs", callback_data="get_pdfs")]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await update.message.reply_text("Welcome! Choose an option:", reply_markup=reply_markup)

# Handle main menu button clicks
async def main_menu_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    if query.data == "ask_questions":
        # Show question buttons
        keyboard = [[InlineKeyboardButton(q, callback_data="qa_" + q)] for q in qa.keys()]
        reply_markup = InlineKeyboardMarkup(keyboard)
        await query.edit_message_text("Select a question:", reply_markup=reply_markup)

    elif query.data == "get_pdfs":
        # Show semester PDF buttons
        keyboard = [[InlineKeyboardButton(sem, callback_data="pdf_" + sem)] for sem in pdf_files.keys()]
        reply_markup = InlineKeyboardMarkup(keyboard)
        await query.edit_message_text("Select your semester and batch:", reply_markup=reply_markup)

# Handle question or pdf buttons
async def qa_pdf_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    data = query.data

    if data.startswith("qa_"):
        question = data[3:]
        answer = qa.get(question, "No answer available.")
        await query.edit_message_text(text=f"Q: {question}\n\nA: {answer}")

    elif data.startswith("pdf_"):
        sem = data[4:]
        file_url = pdf_files.get(sem)
        if file_url:
            # Send PDF file URL as document
            await context.bot.send_document(chat_id=update.effective_chat.id, document=file_url)
        else:
            await query.edit_message_text(text="Sorry, no file found for that selection.")

if __name__ == '__main__':
    import os
    TOKEN = "7823501117:AAH2BkLoeauRvI7K5TqkKTyWP-AKjN-FGzM"  # Replace with your bot token

    app = ApplicationBuilder().token(TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CallbackQueryHandler(main_menu_handler, pattern="^(ask_questions|get_pdfs)$"))
    app.add_handler(CallbackQueryHandler(qa_pdf_handler, pattern="^(qa_|pdf_).+"))

    print("Bot is running...")
    app.run_polling()
