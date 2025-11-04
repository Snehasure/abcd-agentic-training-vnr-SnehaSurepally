# Ollama Code Explainer Bot
An intelligent Telegram bot powered by Ollama and LLMs that can explain, summarize, or optimize code in multiple languages.


## 📘 Table of Contents

Project Overview

Importance of the Project

Project Type & LLM Details

Tools & Technologies Used

System Requirements

System Architecture

Workflow & Flow Diagrams

Implementation Details

Code Explanation

Conclusion


## 🧩 Project Overview

The Ollama Code Explainer Bot is a Python-based Telegram chatbot that leverages local LLMs (Large Language Models) via Ollama to:

🧠 Explain source code (Python, C, Java, etc.)

🪄 Summarize logic and functionality

⚙️ Optimize inefficient code

The bot communicates with Ollama’s REST API (http://localhost:11434/api/generate) and Telegram’s Bot API to deliver interactive, natural-language responses.


## 💡 Importance of the Project

In modern development environments:

Developers frequently need quick explanations for unfamiliar code.

AI-assisted code summarization and optimization reduces debugging time.

Offline/local LLMs (via Ollama) ensure data privacy and cost-free usage — ideal for developers, students, and educators.

Thus, this bot bridges AI explainability and local compute resources, providing a secure and educational coding assistant.


## Project Type & LLM Details

<img width="494" height="265" alt="image" src="https://github.com/user-attachments/assets/b99c18ef-d752-45e6-9a7b-57311dfc66f5" />


## Tools & Technologies Used

<img width="455" height="280" alt="image" src="https://github.com/user-attachments/assets/ee45cf66-cf84-4c43-8e96-66d987b562b8" />


## 💻 System Requirements

OS: Windows / macOS / Linux

RAM: 8 GB minimum (16 GB recommended for LLaMA models)

Python: 3.12+

Installed Ollama app with model llama3.2 or smaller

Telegram Bot Token from @BotFather


## System Architecture

        ┌────────────────────────────┐
        │        User (Telegram)     │
        └────────────┬───────────────┘
                     │
            Message / Code Snippet
                     │
        ┌────────────▼─────────────┐
        │ Telegram Bot (Python)    │
        │ - Receives message       │
        │ - Detects intent (/explain,/optimize,/generate) │
        └────────────┬─────────────┘
                     │
           HTTP POST Request (JSON)
                     │
        ┌────────────▼─────────────┐
        │   Ollama Local Server    │
        │ - Model: llama3.2        │
        │ - Task: Generate text    │
        └────────────┬─────────────┘
                     │
           AI-generated Explanation
                     │
        ┌────────────▼─────────────┐
        │ Telegram Bot (Python)    │
        │ - Formats response       │
        │ - Sends reply to user    │
        └────────────┬─────────────┘
                     │
             Final Response Message
                     │
        ┌────────────▼─────────────┐
        │     User (Telegram)      │
        └──────────────────────────┘


## 🔄 Workflow & Flow Diagrams
🧩 Flow Diagram


<img width="212" height="324" alt="image" src="https://github.com/user-attachments/assets/bef37d12-6c74-4179-888f-b2cc20986d8c" />

<img width="191" height="332" alt="image" src="https://github.com/user-attachments/assets/8a4364f1-2dd5-48e8-8ade-9be31a7a92b1" />


## ⚙️ Implementation Details
### 1. Installing dependencies
!pip install python-telegram-bot aiohttp python-dotenv jupyter


### ✅ Installs the required Python libraries:

python-telegram-bot → for building the Telegram chatbot.

aiohttp → for making asynchronous HTTP requests (used to call Ollama API).

python-dotenv → for loading sensitive information (like tokens and URLs) from .env.

jupyter → used for running this bot in a Jupyter notebook environment.

### 2. Importing modules
import os
import aiohttp
import asyncio
from dotenv import load_dotenv
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes


### ✅ These imports bring in all the required modules:

os – Access environment variables.

aiohttp – Async HTTP client to send POST requests to Ollama.

asyncio – Used for asynchronous programming.

load_dotenv – Loads .env file so that tokens/URLs can be securely accessed.

telegram / telegram.ext – Core Telegram Bot API components for handling messages, commands, etc.

### 3. Loading environment variables
load_dotenv()


### ✅ Loads all key-value pairs from the .env file into the environment.
Example .env file:

TELEGRAM_BOT_TOKEN=123456:ABC-xyz
OLLAMA_URL=http://localhost:11434

### 4. Defining constants
BOT_TOKEN = os.getenv("TELEGRAM_BOT_TOKEN")
OLLAMA_URL = os.getenv("OLLAMA_URL", "http://localhost:11434/api/generate")


### ✅ Reads BOT_TOKEN and OLLAMA_URL from the environment.
If OLLAMA_URL isn’t provided, it defaults to the local Ollama API endpoint.

###5. Querying Ollama model
async def query_ollama(prompt: str) -> str:
    """Send a prompt to Ollama and return response."""
    payload = {
        "model": "llama3.2",
        "prompt": prompt,
        "stream": False
    }


### ✅ This function sends the user’s query (like “Explain my code”) to the Ollama API.

"model": "llama3.2" → model name you’re running locally.

"prompt" → message to send.

"stream": False" → response returned as one block instead of streaming.

    async with aiohttp.ClientSession() as session:
        async with session.post(f"{OLLAMA_URL}/api/generate", json=payload) as resp:


### ✅ Creates an asynchronous HTTP session and sends a POST request to Ollama’s /api/generate endpoint.

            if resp.status == 200:
                try:
                    data = await resp.json()
                    return data.get("response", "⚠ No response from model.")


### ✅ If the request succeeds (200), it parses JSON and returns the model’s response.

                except Exception as e:
                    text = await resp.text()
                    return f"⚠ JSON parse error: {e}\nRaw: {text}"
            else:
                error_text = await resp.text()
                return f"⚠ Ollama error {resp.status}: {error_text}"


### ✅ Handles errors:

If response parsing fails → shows raw output.

If Ollama returns error → shows error message with status code.

6. Start Command
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "🤖 *Welcome to Ollama Code Explainer Bot!*\n\n"
        "Send me any code snippet (Python, C, Java, etc.), and I’ll explain it line-by-line.\n"
        "Use /optimize to get a cleaner or faster version of your code.",
        parse_mode="Markdown"
    )


### ✅ When the user types /start, the bot greets them and provides instructions.

update.message.reply_text() sends a message back to the user.

parse_mode="Markdown" → allows emojis and formatted text.

7. Explaining code
async def explain_code(update: Update, context: ContextTypes.DEFAULT_TYPE):
    code = update.message.text.strip()
    if not code:
        await update.message.reply_text("⚠️ Please send valid code.")
        return


### ✅ This function is triggered when a user sends a non-command text message.
If message is empty, it alerts the user.

    await update.message.reply_text("💭 Analyzing your code with Ollama, please wait...")


### ✅ Sends a waiting message while processing.

    prompt = f"Explain the following {len(code.splitlines())}-line code step-by-step:\n\n{code}"
    explanation = await query_ollama(prompt)


### ✅ Builds a natural-language prompt to instruct Ollama to explain the code line by line.

    await update.message.reply_text(f"🧩 *Explanation:*\n{explanation[:4000]}", parse_mode="Markdown")


### ✅ Sends the model’s explanation (trimmed to Telegram’s max message size).

### 8. Optimizing code
async def optimize_code(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        await update.message.reply_text("Please send code after /optimize command.")
        return


### ✅ The /optimize command allows users to send code to improve readability or performance.

    code = " ".join(context.args)
    await update.message.reply_text("⚙️ Optimizing your code...")


### ✅ Joins all arguments into one string and notifies the user.

    prompt = f"Optimize the following code for readability and performance:\n\n{code}\n\nAlso explain the changes briefly."
    optimized = await query_ollama(prompt)


### ✅ Creates a prompt instructing the LLM to refactor the code and explain the improvements.

    await update.message.reply_text(f"🚀 *Optimized Version:*\n{optimized[:4000]}", parse_mode="Markdown")


### ✅ Sends back the optimized version and explanation.

###9. Building the bot
app = ApplicationBuilder().token(BOT_TOKEN).build()


### ✅ Creates the main bot application and links your Telegram bot token.

10. Adding handlers
app.add_handler(CommandHandler("start", start))
app.add_handler(CommandHandler("optimize", optimize_code))
app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, explain_code))


### ✅ Registers functions that respond to commands or messages:

/start → start()

/optimize → optimize_code()

Text messages → explain_code()

###11. Running the bot
import nest_asyncio
nest_asyncio.apply()


### ✅ Allows the bot to run properly inside environments like Jupyter that already have an event loop.

print("🤖 Bot is running... Press Ctrl+C to stop.")
await app.run_polling()


### ✅ Starts polling Telegram servers continuously for new messages and processes them in real time.

## 🌐 Deployment on Render (Free Hosting Option)

The Ollama Code Explainer Bot can be deployed online so that anyone can interact with it on Telegram without running the

backend locally.

For this purpose, we use Render.com, a free and beginner-friendly cloud hosting platform that runs Python applications 

continuously.

⚙️ What We Are Doing

We host our Python-based Telegram bot (the backend code) on Render’s cloud infrastructure.

This allows the bot to stay online 24/7, listening to user messages and responding instantly — even when our personal system

is turned off.

Render runs the bot script (bot.py) automatically and keeps it alive using a web service setup.

The bot still needs to connect to the Ollama model server (which generates code explanations and optimizations).

Since Ollama currently runs locally (on our computer), we use Ngrok to expose it to the internet securely.

🧩 Steps Performed

Uploaded Code to GitHub:

The entire bot project was pushed to GitHub so that Render can fetch it directly.

Connected GitHub Repository to Render:

On Render, we created a new Web Service and linked it to the GitHub repository.

Configured the Build and Run Commands:

Runtime: Python 3

Build Command:

pip install -r requirements.txt


Start Command:

python bot.py


Added Environment Variables (Secrets):

TELEGRAM_BOT_TOKEN=your_bot_token

OLLAMA_URL=https://a1b2c3d4.ngrok.io

MODEL=llama3.2


These variables securely store sensitive data like API tokens and URLs.

Deployed the Service:

Once deployed, Render automatically installs dependencies and starts the bot process.

🌐 Connecting Ollama with Render (Using Ngrok)

Since Render cannot run Ollama locally, we make our local Ollama instance accessible via a secure tunnel using Ngrok:

ngrok http 11434


This generates a temporary public URL like:

https://a1b2c3d4.ngrok.io


We then assign this URL to OLLAMA_URL so the hosted bot can reach our local LLM for processing user requests.

💡 Importance of This Deployment

✅ Global Accessibility:

Once deployed, anyone on Telegram can chat with your bot from anywhere in the world — no setup required.

✅ Continuous Availability:

Render keeps your bot running even if your computer is offline.

✅ Secure & Scalable:

Environment variables protect sensitive keys, and the service can scale automatically with user demand.

✅ Educational & Practical:

Demonstrates integration of cloud deployment + local AI inference, bridging local computation (Ollama) and public chatbot

access (Telegram).


## 🧩 Conclusion

The Ollama Code Explainer Bot showcases how local AI (LLMs) can power intelligent assistants for developers — providing 

explainability, performance tuning, and generation capabilities all from within Telegram.

This project highlights:

The utility of integrating local AI models with real-world chat interfaces.

The potential of open-source LLMs for privacy-preserving AI solutions.

The synergy between Python, Ollama, and Telegram APIs for seamless interaction.

## Presentation Link

https://www.canva.com/design/DAG3ty2bD6Q/yuFs_iP8EvtjEKsRvHqvGw/edit?utm_content=DAG3ty2bD6Q&utm_campaign=designshare&utm_medium=link2&utm_source=sharebutton

## Video Link

https://1drv.ms/v/c/a1fbdbf99dcec386/Eele3zOx6hZIrAaK9oQjJGYBirD6QagUjK1VSkYuI1lRIQ?e=LrzQdy
