import discord
from discord.ext import commands, tasks
import asyncio
import csv
from datetime import datetime, timedelta

import os
from dotenv import load_dotenv

load_dotenv()
TOKEN = os.getenv("TOKEN")  # Replace with your bot token
bot = commands.Bot(command_prefix="!", intents=discord.Intents.all())

# Database placeholders
predictions = {}  # {user_id: {round: {match_id: {'team': prediction, 'try_scorers': [scorer1, scorer2]}}}}
scores = {}  # {user_id: total_points}
round_results = {}  # {round: {match_id: {'winner': result, 'try_scorers': [scorer1, scorer2]}}}
leaderboard = []  # [(user_id, points)]
match_data = {}  # {round: {match_id: {'teams': [team1, team2], 'eligible_scorers': {team1: [...], team2: [...]}}}}

def is_admin():
    def predicate(ctx):
        return ctx.author.guild_permissions.administrator
    return commands.check(predicate)

@bot.event
async def on_ready():
    print(f'Logged in as {bot.user}')
    reminder_loop.start()

@bot.command()
@is_admin()
async def upload_csv(ctx):
    """Handle CSV uploads containing match data and eligible try scorers (Admin only)."""
    if not ctx.message.attachments:
        await ctx.send("Please attach a CSV file.")
        return
    
    attachment = ctx.message.attachments[0]
    file_content = await attachment.read()
    
    decoded_content = file_content.decode("utf-8").splitlines()
    reader = csv.reader(decoded_content)
    next(reader)  # Skip header
    
    for row in reader:
        round_num, match_id, team1, team2, scorers1, scorers2 = row
        
        if round_num not in match_data:
            match_data[round_num] = {}
        match_data[round_num][match_id] = {
            'teams': [team1, team2],
            'eligible_scorers': {
                team1: scorers1.split(";"),
                team2: scorers2.split(";")
            }
        }
    
    await ctx.send("Match data successfully uploaded.")

@bot.command()
async def predict(ctx, round_num: int, match_id: str, team: str, scorer1: str, scorer2: str):
    """Users submit their predictions including try scorers."""
    user_id = ctx.author.id
    if user_id not in predictions:
        predictions[user_id] = {}
    if round_num not in predictions[user_id]:
        predictions[user_id][round_num] = {}
    predictions[user_id][round_num][match_id] = {'team': team, 'try_scorers': [scorer1, scorer2]}
    await ctx.send(f'Prediction recorded: {team} for match {match_id} (Round {round_num}), Try scorers: {scorer1}, {scorer2}')

@bot.command()
async def submit_results(ctx, round_num: int, match_id: str, result: str, scorer1: str, scorer2: str):
    """Admins submit match results including try scorers."""
    if round_num not in round_results:
        round_results[round_num] = {}
    round_results[round_num][match_id] = {'winner': result, 'try_scorers': [scorer1, scorer2]}
    await ctx.send(f'Result recorded: {result} for match {match_id} (Round {round_num}), Try scorers: {scorer1}, {scorer2}')
    await calculate_scores(round_num)

async def calculate_scores(round_num):
    """Calculate user scores based on results."""
    global leaderboard
    for user_id, rounds in predictions.items():
        if round_num in rounds:
            for match_id, picks in rounds[round_num].items():
                if match_id in round_results[round_num]:
                    correct_team = round_results[round_num][match_id]['winner'] == picks['team']
                    points = 4 if picks['team'] == "draw" and correct_team else 1 if correct_team else 0
                    correct_scorers = set(picks['try_scorers']) & set(round_results[round_num][match_id]['try_scorers'])
                    points += len(correct_scorers)  # 1 point per correct try scorer
                    scores[user_id] = scores.get(user_id, 0) + points
    leaderboard = sorted(scores.items(), key=lambda x: x[1], reverse=True)

@bot.command()
async def leaderboard(ctx):
    """Show the top 10 users."""
    top_10 = leaderboard[:10]
    message = "Leaderboard:\n" + "\n".join([f'{idx+1}. <@{user_id}> - {points} pts' for idx, (user_id, points) in enumerate(top_10)])
    await ctx.send(message)

@bot.command()
async def my_rank(ctx):
    """Users check their own rank and points."""
    user_id = ctx.author.id
    rank = next((i+1 for i, (uid, _) in enumerate(leaderboard) if uid == user_id), None)
    points = scores.get(user_id, 0)
    await ctx.send(f'Your rank: {rank}, Points: {points}')

@tasks.loop(minutes=1)
async def reminder_loop():
    """Send prediction reminders."""
    now = datetime.utcnow()
    # Placeholder for checking match times and sending reminders
    
bot.run(TOKEN)
