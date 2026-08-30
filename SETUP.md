# PixEdge Setup Guide - Krama Project

## Overview

PixEdge is an ultra-fast edge media hosting platform that uses Telegram as a free, unlimited storage backend. This guide covers the setup for the Krama project with Neon PostgreSQL authentication.

## Architecture

```
┌─────────────┐      ┌──────────────┐      ┌─────────────┐
│   Client    │─────▶│  Next.js     │─────▶│   Neon      │
│  (Browser)  │      │  (Vercel)    │      │  PostgreSQL │
└─────────────┘      └──────┬───────┘      └─────────────┘
                            │
                     ┌──────▼───────┐      ┌─────────────┐
                     │  Upstash     │      │  Telegram   │
                     │  Redis       │      │  (Storage)  │
                     └──────────────┘      └─────────────┘
```

## Prerequisites

1. **Neon PostgreSQL Account** - https://console.neon.tech
2. **Telegram Bot Token** - From @BotFather
3. **Telegram Storage Channel** - Create a private channel
4. **Upstash Redis Account** - For media metadata
5. **Vercel Account** - For deployment

## Step 1: Create Neon PostgreSQL Database

1. Go to https://console.neon.tech and create a new project
2. Copy the connection string (it looks like: `postgresql://user:password@host/database?sslmode=require`)
3. Run the database schema in `prisma/schema.sql`:
   - Go to Neon Console → SQL Editor
   - Paste and execute the contents of `prisma/schema.sql`

## Step 2: Configure Telegram

### Bot Setup
1. Open @BotFather on Telegram
2. Send `/newbot` and follow prompts
3. Copy the bot token

### Storage Channel
1. Create a private channel for media storage
2. Add your bot as an admin with these permissions:
   - Post messages
   - Edit messages
   - Delete messages
3. Get the channel ID:
   - Forward a message from channel to @userinfobot
   - Or use the API: `https://api.telegram.org/bot<TOKEN>/getUpdates`

## Step 3: Get OAuth Credentials

### Google OAuth
1. Go to https://console.cloud.google.com
2. Create credentials → OAuth 2.0 Client ID
3. Add authorized redirect URI: `https://your-domain.com/api/auth/callback/google`

### GitHub OAuth
1. Go to https://github.com/settings/developers
2. New OAuth App
3. Homepage URL: `https://your-domain.com`
4. Callback URL: `https://your-domain.com/api/auth/callback/github`

## Step 4: Set Up Upstash Redis

1. Create database at https://console.upstash.com
2. Copy REST URL and Token from the dashboard

## Step 5: Configure Environment Variables

Create a `.env.local` file in the project root:

```env
# Telegram Bot (Required)
TELEGRAM_BOT_TOKEN=your_bot_token
TELEGRAM_CHAT_ID=your_channel_id
TELEGRAM_LOG_CHANNEL_ID=your_channel_id

# Telegram Bot Info
NEXT_PUBLIC_TELEGRAM_BOT_USERNAME=your_bot_username

# Upstash Redis (Required for media metadata)
UPSTASH_REDIS_REST_URL=https://your-redis-url.upstash.io
UPSTASH_REDIS_REST_TOKEN=your_redis_token

# Neon PostgreSQL (Required for authentication)
DATABASE_URL=postgresql://user:pass@host/db?sslmode=require

# OAuth Providers
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
GITHUB_ID=your_github_client_id
GITHUB_SECRET=your_github_client_secret

# MTProto (Optional - for 2GB uploads)
TELEGRAM_API_ID=your_api_id
TELEGRAM_API_HASH=your_api_hash
TELEGRAM_SESSION_STRING=your_session_string
MAX_UPLOAD_SIZE_MB=2000
```

## Step 6: Generate MTProto Session (Optional)

For 2GB upload support:

```bash
npm install
npm run generate:session
# Follow prompts with your phone number and OTP
# Copy the printed TELEGRAM_SESSION_STRING
```

## Step 7: Deploy to Vercel

### Option 1: One-Click Deploy

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=YOUR_REPO_URL)

### Option 2: Manual Deploy

```bash
# Install Vercel CLI
npm i -g vercel

# Login
vercel login

# Deploy
vercel

# Set environment variables in Vercel dashboard
vercel env add DATABASE_URL
vercel env add TELEGRAM_BOT_TOKEN
# ... add all other variables
```

### Option 3: GitHub Integration

1. Push to GitHub
2. Go to Vercel Dashboard → New Project
3. Import from GitHub
4. Configure environment variables in project settings

## Step 8: Configure Telegram Webhook

After deployment, set up the webhook:

```
https://api.telegram.org/bot<YOUR_BOT_TOKEN>/setWebhook?url=https://your-domain.com/api/webhook/telegram
```

## Step 9: Verify Setup

1. Visit your deployed URL
2. Try logging in with email/password
3. Try OAuth login (Google/GitHub)
4. Upload a test image
5. Check Telegram channel for stored media

## Environment Variables Reference

| Variable | Required | Description |
|----------|----------|-------------|
| `DATABASE_URL` | Yes | Neon PostgreSQL connection string |
| `TELEGRAM_BOT_TOKEN` | Yes | Bot token from @BotFather |
| `TELEGRAM_CHAT_ID` | Yes | Storage channel ID |
| `UPSTASH_REDIS_REST_URL` | Yes | Redis REST URL |
| `UPSTASH_REDIS_REST_TOKEN` | Yes | Redis REST Token |
| `GOOGLE_CLIENT_ID` | No | For Google OAuth |
| `GOOGLE_CLIENT_SECRET` | No | For Google OAuth |
| `GITHUB_ID` | No | For GitHub OAuth |
| `GITHUB_SECRET` | No | For GitHub OAuth |
| `TELEGRAM_API_ID` | No | For MTProto (2GB uploads) |
| `TELEGRAM_API_HASH` | No | For MTProto |
| `TELEGRAM_SESSION_STRING` | No | For MTProto |

## Troubleshooting

### "Database connection failed"
- Verify Neon DATABASE_URL is correct
- Check SSL mode is enabled (`?sslmode=require`)
- Ensure Neon project is active

### "Telegram upload failed"
- Verify bot token is correct
- Ensure bot is admin in storage channel
- Check channel ID format (use negative numbers for private channels)

### "OAuth not working"
- Verify callback URLs match exactly
- Check client ID/secret are correct
- Ensure authorized domains are configured

### "Redis connection failed"
- Verify Upstash URL and token
- Check if Redis instance is active

## Production Checklist

- [ ] Enable SSL on Neon database
- [ ] Set up Telegram bot commands
- [ ] Configure authorized domains in OAuth apps
- [ ] Set up monitoring/analytics
- [ ] Enable Vercel Analytics
- [ ] Configure rate limiting

## File Structure

```
PixEdge/
├── prisma/
│   ├── schema.prisma      # Prisma schema
│   └── schema.sql         # Raw SQL schema
├── src/
│   ├── app/
│   │   ├── api/
│   │   │   ├── auth/     # Auth endpoints
│   │   │   ├── upload/   # Upload endpoint
│   │   │   └── ...
│   │   └── ...
│   └── lib/
│       ├── auth.ts       # NextAuth config
│       ├── db.ts         # Redis operations
│       ├── gramjs.ts     # MTProto operations
│       └── telegram.ts   # Bot API operations
├── .env.local            # Environment variables
├── vercel.json           # Vercel config
└── package.json
```

## Support

- GitHub Issues: https://github.com/GeekLuffy/PixEdge/issues
- Telegram Channel: [@EdgeBots](https://t.me/EdgeBots)
- Support Chat: [@EdgeBotSupport](https://t.me/EdgeBotSupport)
