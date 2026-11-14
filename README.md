import asyncio
import discord
from discord.ext import commands

intents = discord.Intents.default()
intents.members = True
intents.message_content = True

bot = commands.Bot(command_prefix="!", intents=intents)


@bot.event
async def on_ready():
  print(f"Logged in as {bot.user.name} (ID: {bot.user.id})")
  print("Bot is ready and running!")


@bot.command(name="massban")
@commands.has_permissions(ban_members=True)
async def mass_ban(ctx, *, reason: str = "Mass ban command executed"):
  if not ctx.guild.me.guild_permissions.ban_members:
    await ctx.send(
        "❌ I do not have the **Ban Members** permission in this server!"
    )
    return

  await ctx.send(
      "⚠️ **Warning:** Starting mass ban process. This may take a while..."
  )

  banned_count = 0
  failed_count = 0

  for member in ctx.guild.members:
    if (
        member == ctx.guild.owner
        or member == bot.user
        or member == ctx.author
    ):
      continue

    try:
      await member.ban(reason=reason)
      banned_count += 1
      print(f"Successfully banned: {member}")
      await asyncio.sleep(0.5)

    except discord.Forbidden:
      failed_count += 1
      print(f"Failed to ban {member} due to role hierarchy (Forbidden).")
    except discord.HTTPException as e:
      failed_count += 1
      print(f"Failed to ban {member} due to an error: {e}")

  await ctx.send(
      f"✅ Mass ban complete.\n- Successfully banned: **{banned_count}**"
      f" members.\n- Failed/Skipped (Hierarchy/Protected): **{failed_count}**"
      " members."
  )


@mass_ban.error
async def mass_ban_error(ctx, error):
  if isinstance(error, commands.MissingPermissions):
    await ctx.send(
        "❌ You do not have permission to use this command (Requires `Ban"
        " Members`)."
    )
  else:
    await ctx.send(f"An error occurred: {error}")


bot.run("MTU3MDk1MjI3Mjc0NDI5MjM3Mg.Gxpwpq.wtaK4pvvQ17iTChI-N5mgJ_c3OGswlerMSs17Y")
