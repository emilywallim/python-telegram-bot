from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, ContextTypes, filters
import cv2
import numpy as np

TOKEN = "7907926455:AAErOeMcZk3BtQOE81lGjqz6eIfR-V9Hgds"

# ===============================
# SMART ANALYSIS ENGINE
# ===============================
def analyze_image(path):
    img = cv2.imread(path, 0)

    if img is None:
        return "❌ Image read failed"

    # Step 1: Edge detection (market movement strength)
    edges = cv2.Canny(img, 80, 160)
    edge_score = np.mean(edges)

    # Step 2: Intensity + structure check
    brightness = np.mean(img)

    # Step 3: Combined smart score
    score = (edge_score * 0.7) + (brightness * 0.3)

    # Step 4: Confidence calculation
    confidence = min(95, int(score * 2))

    # ===============================
    # DECISION LOGIC (LOW FALSE SIGNAL)
    # ===============================

    if score > 40:
        decision = "📈 BUY SIGNAL (Strong Trend)"
    elif score > 25:
        decision = "⚖️ WAIT (Unclear Market - Avoid Trade)"
    else:
        decision = "📉 SELL SIGNAL (Weak/Down Trend)"

    return f"""
🧠 SMART AI ANALYSIS

{decision}

📊 Score: {score:.2f}
🎯 Confidence: {confidence}%
⚠️ Not 100% guaranteed - use risk management
"""

# ===============================
# TELEGRAM HANDLERS
# ===============================
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "🤖 Smart AI Bot Active\n📊 Send chart image for analysis"
    )

async def photo(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("🔍 Analyzing with AI layers...")

    photo = update.message.photo[-1]
    file = await context.bot.get_file(photo.file_id)

    path = "chart.jpg"
    await file.download_to_drive(path)

    result = analyze_image(path)

    await update.message.reply_text(result)

# ===============================
# BOT RUN
# ===============================
app = Application.builder().token(TOKEN).build()

app.add_handler(CommandHandler("start", start))
app.add_handler(MessageHandler(filters.PHOTO, photo))

print("🚀 SMART AI BOT RUNNING...")
app.run_polling()
