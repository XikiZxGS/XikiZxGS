
import discord
from discord.ext import commands, tasks
import asyncio
from discord.ext import commands
from datetime import timedelta

intents = discord.Intents.all()
intents.typing = False
intents.presences = False

bot = commands.Bot(command_prefix="!", intents=intents)

@bot.event
async def on_command_error(ctx, error):
    if isinstance(error, commands.CommandNotFound):
        await ctx.send("VN: Không có lệnh này trong mã code! EN: There is no such command in the code!")
    elif isinstance(error, commands.MissingRequiredArgument):
        await ctx.send("VN: Thiếu lệnh! EN: Wrong command!")
    elif isinstance(error, commands.MissingPermissions):
        await ctx.send("VN: Bạn thiếu quyền để thực hiện lệnh này EN: You don't have permission to use this command!")
    elif isinstance(error, commands.BotMissingPermissions):
        await ctx.send("VN: Bot thiếu quyền để thực hiện lệnh này.EN: I don't have permission to use this command!")
    else:
        await ctx.send("VN: Đã xảy ra lỗi khi thực hiện lệnh. Vui lòng thử lại sau. EN: There was an error while executing the command. Please try again later.")

@bot.event
async def on_ready():
    print('Code Running Now')
    await bot.change_presence(activity=discord.Game(name="RuteX | Community"))

@bot.event
async def help(ctx):
    help_message = "Help Menu:\n"
    for command in bot.commands:
        if command.help:
            help_message += f"{command.name}: {command.help}\n"
    await ctx.send(help_message)

@bot.command()
async def check(ctx):
    server_list = "\n".join(guild.name for guild in bot.guilds)
    embed = discord.Embed(title="Servers", description=server_list, color=discord.Color.blurple())
    await ctx.send(embed=embed)

@bot.command()
async def mute(ctx, member: discord.Member, duration: str = None):
    mute_role = discord.utils.get(ctx.guild.roles, name="Muted")
    if not mute_role:
        await ctx.send("Mute role not found. Please create a 'Muted' role and try again.")
        return

    try:
        await member.add_roles(mute_role)
        await ctx.send(f"{member.mention} has been muted.")
        
        if duration:
            duration_parts = duration.split(' ')
            time_units = {'d': 86400, 'h': 3600, 'm': 60}
            mute_duration = sum(int(amount) * time_units[unit] for amount, unit in zip(duration_parts[0::2], duration_parts[1::2]))
            
            # Implement your code here to handle the mute duration calculation as needed
            # For example, you can add a task to schedule an unmute after the specified duration
            
    except discord.Forbidden:
        await ctx.send("I don't have permission to mute this member.")

@bot.command()
@commands.has_permissions(manage_roles=True)
async def unmute(ctx, member: discord.Member):
    mute_role = discord.utils.get(ctx.guild.roles, name="Muted")
    if not mute_role:
        await ctx.send("Mute role not found. Please create a 'Muted' role and try again.")
        return

    try:
        await member.remove_roles(mute_role)
        await ctx.send(f"{member.mention} has been unmuted.")
    except discord.Forbidden:
        await ctx.send("I don't have permission to unmute this member.")

@bot.command()
@commands.has_permissions(ban_members=True)
async def ban(ctx, member: discord.Member, reason=None):
    try:
        await member.ban(reason=reason)
        await ctx.send(f"{member.mention} has been banned.")
    except discord.Forbidden:
        await ctx.send("I don't have permission to ban this member.")

@bot.command()
@commands.has_permissions(ban_members=True)
async def unban(ctx, *, user_id: int):
    try:
        user = await bot.fetch_user(user_id)
        await ctx.guild.unban(user)
        await ctx.send(f"{user.mention} has been unbanned.")
    except discord.Forbidden:
        await ctx.send("I don't have permission to unban this user.")

@bot.command()
async def nuke(ctx):
    if discord.utils.get(ctx.author.roles, id=1160171772256194691) is None:
        return
    await ctx.channel.purge(limit=None)
    await ctx.send(f"Nuked by {ctx.author.mention}")

@bot.command()
async def kick(ctx, member_id: int, *, reason=None):
    if ctx.author.guild_permissions.kick_members:
        try:
            member = await bot.fetch_user(member_id)
            await ctx.guild.kick(member, reason=reason)
            await ctx.send(f"Member with ID {member_id} has been kicked.")
            print(f"Member with ID {member_id} has been kicked.")
        except discord.NotFound:
            await ctx.send("Member not found.")
        except discord.Forbidden:
            await ctx.send("I don't have permission to kick members.")
    else:
        await ctx.send("You don't have permission to use this command.")

bot.run("")
