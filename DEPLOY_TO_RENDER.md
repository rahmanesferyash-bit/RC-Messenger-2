# Deploying Race Corps Bot to Render via GitHub

## Step 1 — Get a free PostgreSQL database (Neon)

1. Go to https://neon.tech and sign up for free
2. Create a new project (any name, any region)
3. Click "Connection Details" and copy the **Connection string**
   - It looks like: `postgresql://user:password@ep-xxx.us-east-2.aws.neon.tech/neondb?sslmode=require`
4. Keep this tab open — you will need this string in Step 4

---

## Step 2 — Push this bot to GitHub

1. Go to https://github.com/new and create a **private** repository called `racecorps-bot`
2. Extract the `.tar.gz` file you downloaded from Replit
3. Open a terminal in the extracted folder and run:
   ```
   git init
   git add .
   git commit -m "initial bot upload"
   git remote add origin https://github.com/YOUR_USERNAME/racecorps-bot.git
   git push -u origin main
   ```
   Replace YOUR_USERNAME with your GitHub username.

> ⚠️ IMPORTANT: Never put your bot token or database password inside any file
> that you commit to GitHub. Only set them in the Render dashboard (Step 4).

---

## Step 3 — Connect to Render

1. Go to https://render.com and sign up / log in (free)
2. Click **"New +"** → **"Blueprint"**
3. Connect your GitHub account if prompted
4. Select the `racecorps-bot` repository
5. Render will detect the `render.yaml` file automatically
6. Click **"Apply"**

---

## Step 4 — Set your environment variables (IMPORTANT)

After Render creates the service, go to:
**Dashboard → racecorps-bot → Environment**

Add these two variables:

| Key                  | Value                                      |
|----------------------|--------------------------------------------|
| `DISCORD_BOT_TOKEN`  | Your token from discord.com/developers     |
| `DATABASE_URL`       | The Neon connection string from Step 1     |

Click **Save Changes** — Render will redeploy automatically.

---

## Step 5 — Set up the database tables

Run this once in the Neon SQL editor (https://console.neon.tech → SQL Editor):

```sql
CREATE TABLE IF NOT EXISTS guild_settings (
  guild_id TEXT PRIMARY KEY, prefix TEXT NOT NULL DEFAULT '!',
  mod_log_channel_id TEXT, server_log_channel_id TEXT,
  welcome_channel_id TEXT, welcome_message TEXT,
  leave_channel_id TEXT, leave_message TEXT, autorole_id TEXT,
  level_up_message TEXT DEFAULT 'GG {user}, you just advanced to **level {level}**!',
  level_up_channel_id TEXT, starboard_channel_id TEXT,
  starboard_threshold INTEGER NOT NULL DEFAULT 3,
  leveling_enabled BOOLEAN NOT NULL DEFAULT TRUE
);
CREATE TABLE IF NOT EXISTS user_levels (
  guild_id TEXT NOT NULL, user_id TEXT NOT NULL,
  xp INTEGER NOT NULL DEFAULT 0, level INTEGER NOT NULL DEFAULT 0,
  total_messages INTEGER NOT NULL DEFAULT 0, last_message_at TIMESTAMP,
  PRIMARY KEY (guild_id, user_id)
);
CREATE TABLE IF NOT EXISTS level_rewards (
  guild_id TEXT NOT NULL, level INTEGER NOT NULL, role_id TEXT NOT NULL,
  PRIMARY KEY (guild_id, level)
);
CREATE TABLE IF NOT EXISTS level_ignored_channels (
  guild_id TEXT NOT NULL, channel_id TEXT NOT NULL,
  PRIMARY KEY (guild_id, channel_id)
);
CREATE TABLE IF NOT EXISTS warnings (
  id SERIAL PRIMARY KEY, guild_id TEXT NOT NULL, user_id TEXT NOT NULL,
  moderator_id TEXT NOT NULL, reason TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);
CREATE TABLE IF NOT EXISTS automod_settings (
  guild_id TEXT PRIMARY KEY,
  anti_invite BOOLEAN NOT NULL DEFAULT FALSE, anti_link BOOLEAN NOT NULL DEFAULT FALSE,
  anti_spam BOOLEAN NOT NULL DEFAULT FALSE, anti_caps BOOLEAN NOT NULL DEFAULT FALSE,
  anti_mass_mention BOOLEAN NOT NULL DEFAULT FALSE,
  caps_threshold INTEGER NOT NULL DEFAULT 70, caps_min_length INTEGER NOT NULL DEFAULT 10,
  mention_threshold INTEGER NOT NULL DEFAULT 5, spam_count INTEGER NOT NULL DEFAULT 5,
  spam_window_ms INTEGER NOT NULL DEFAULT 5000
);
CREATE TABLE IF NOT EXISTS reaction_roles (
  guild_id TEXT NOT NULL, channel_id TEXT NOT NULL, message_id TEXT NOT NULL,
  emoji TEXT NOT NULL, role_id TEXT NOT NULL, PRIMARY KEY (message_id, emoji)
);
CREATE TABLE IF NOT EXISTS custom_commands (
  guild_id TEXT NOT NULL, name TEXT NOT NULL, response TEXT NOT NULL,
  PRIMARY KEY (guild_id, name)
);
CREATE TABLE IF NOT EXISTS tags (
  guild_id TEXT NOT NULL, name TEXT NOT NULL, content TEXT NOT NULL,
  author_id TEXT NOT NULL, created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  PRIMARY KEY (guild_id, name)
);
CREATE TABLE IF NOT EXISTS starboard_messages (
  guild_id TEXT NOT NULL, original_message_id TEXT NOT NULL,
  starboard_message_id TEXT NOT NULL, star_count INTEGER NOT NULL DEFAULT 0,
  PRIMARY KEY (guild_id, original_message_id)
);
CREATE TABLE IF NOT EXISTS afk_users (
  guild_id TEXT NOT NULL, user_id TEXT NOT NULL,
  message TEXT NOT NULL, since TIMESTAMP NOT NULL DEFAULT NOW(),
  PRIMARY KEY (guild_id, user_id)
);
CREATE TABLE IF NOT EXISTS ticket_settings (
  guild_id TEXT PRIMARY KEY, category_id TEXT, staff_role_id TEXT,
  log_channel_id TEXT, panel_channel_id TEXT, panel_message_id TEXT,
  open_message TEXT DEFAULT 'Thanks for opening a ticket! Staff will be with you shortly.'
);
CREATE TABLE IF NOT EXISTS tickets (
  id SERIAL PRIMARY KEY, guild_id TEXT NOT NULL, user_id TEXT NOT NULL,
  channel_id TEXT NOT NULL UNIQUE, status TEXT NOT NULL DEFAULT 'open',
  claimed_by TEXT, created_at TIMESTAMP NOT NULL DEFAULT NOW(), closed_at TIMESTAMP
);
CREATE TABLE IF NOT EXISTS giveaways (
  id SERIAL PRIMARY KEY, guild_id TEXT NOT NULL, channel_id TEXT NOT NULL,
  message_id TEXT NOT NULL UNIQUE, prize TEXT NOT NULL,
  winners_count INTEGER NOT NULL DEFAULT 1, host_id TEXT NOT NULL,
  ends_at TIMESTAMP NOT NULL, ended BOOLEAN NOT NULL DEFAULT FALSE,
  entrants TEXT[] NOT NULL DEFAULT '{}', winner_ids TEXT[] NOT NULL DEFAULT '{}'
);
```

---

## Step 6 — Verify

- In Render dashboard → **racecorps-bot → Logs**
- You should see: `✅ Logged in as Race Corps Messenger#XXXX`
- Your bot will now stay online 24/7 for free

---

## Where to find your bot token

1. Go to https://discord.com/developers/applications
2. Select your application → **Bot** tab
3. Click **"Reset Token"** → copy the new token
4. Paste it into Render as `DISCORD_BOT_TOKEN`

> Never share or commit your bot token to GitHub. Keep it only in the Render dashboard.
