# Bot
import discord
from discord.ext import commands
import json
import os
import asyncio

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(command_prefix="!", intents=intents)

# -------------------- FILE SETUP --------------------

FILES = {
    "tasks": "tasks.json",
    "attendance": "attendance.json",
    "sessions": "sessions.json",
    "users": "users.json",
    "forms": "forms.json"
}

# Create files if not exist
for key in FILES:
    if not os.path.exists(FILES[key]):
        with open(FILES[key], "w") as f:
            json.dump({}, f)

def load_data(file):
    with open(file, "r") as f:
        return json.load(f)

def save_data(file, data):
    with open(file, "w") as f:
        json.dump(data, f, indent=4)

# -------------------- BOT READY --------------------

@bot.event
async def on_ready():
    print(f"Logged in as {bot.user}")

# -------------------- BASIC TEST --------------------

@bot.command()
async def ping(ctx):
    await ctx.send("Bot is working 🚀")

# -------------------- TASK SYSTEM --------------------

@bot.command()
async def addtask(ctx, *, task):
    data = load_data(FILES["tasks"])
    user = str(ctx.author.id)

    if user not in data:
        data[user] = []

    data[user].append({"task": task, "status": "Pending"})
    save_data(FILES["tasks"], data)

    await ctx.send("Task added successfully ✅")

@bot.command()
async def mytasks(ctx):
    data = load_data(FILES["tasks"])
    user = str(ctx.author.id)

    if user not in data or len(data[user]) == 0:
        await ctx.send("No tasks found.")
    else:
        msg = ""
        for i, t in enumerate(data[user]):
            msg += f"{i+1}. {t['task']} - {t['status']}\n"
        await ctx.send(msg)

@bot.command()
async def donetask(ctx, number: int):
    data = load_data(FILES["tasks"])
    user = str(ctx.author.id)

    if user in data and 0 < number <= len(data[user]):
        data[user][number-1]["status"] = "Completed"
        save_data(FILES["tasks"], data)
        await ctx.send("Task marked as completed 🎉")
    else:
        await ctx.send("Invalid task number.")

# -------------------- REMINDER --------------------

@bot.command()
async def remind(ctx, minutes: int, *, message):
    await ctx.send(f"Reminder set for {minutes} minutes ⏰")
    await asyncio.sleep(minutes * 60)
    await ctx.send(f"Reminder: {message}")

# -------------------- ATTENDANCE --------------------

@bot.command()
async def checkin(ctx):
    data = load_data(FILES["attendance"])
    user = str(ctx.author.id)

    data[user] = "Present"
    save_data(FILES["attendance"], data)

    await ctx.send("Attendance marked ✅")

# -------------------- SESSION BOOKING --------------------

@bot.command()
async def createsession(ctx, *, session_name):
    data = load_data(FILES["sessions"])
    session_id = str(len(data) + 1)

    data[session_id] = {
        "name": session_name,
        "participants": []
    }

    save_data(FILES["sessions"], data)
    await ctx.send(f"Session created with ID {session_id}")

@bot.command()
async def book(ctx, session_id: str):
    data = load_data(FILES["sessions"])

    if session_id in data:
        data[session_id]["participants"].append(str(ctx.author.id))
        save_data(FILES["sessions"], data)
        await ctx.send("Session booked successfully 📅")
    else:
        await ctx.send("Invalid session ID")

# -------------------- FORM REMINDER --------------------

@bot.command()
async def formremind(ctx):
    await ctx.send("Please submit the form and reply DONE when finished.")

@bot.event
async def on_message(message):
    if message.author == bot.user:
        return

    if message.content.upper() == "DONE":
        await message.channel.send("Form submission confirmed ✅")

    await bot.process_commands(message)

# -------------------- USER REGISTRATION --------------------

@bot.command()
async def register(ctx, profile_id):
    data = load_data(FILES["users"])
    data[str(ctx.author.id)] = profile_id
    save_data(FILES["users"], data)

    await ctx.send("User profile linked successfully 🔗")

# -------------------- RUN BOT --------------------
import os
bot.run(os.getenv("MTQ3MDc3NTM2NjgzMzY3MjMzNg.Gq6bs0.ezBtAEJux5a9_1T3NrSNu-eAn0SvxJhWz5kC9A"))
