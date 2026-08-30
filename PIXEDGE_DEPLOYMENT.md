# PixEdge Deployment Guide

Complete deployment guide for PixEdge media hosting platform, built for the Krama project.

## Overview

PixEdge is an ultra-fast edge media hosting platform that uses Telegram as a free, unlimited storage backend. It's designed to handle large media files (up to 2GB with MTProto) for the Krama messaging app.

```
Architecture:
┌─────────────┐      ┌──────────────┐      ┌─────────────┐
│   Client    │─────▶│  Next.js     │─────▶│   Neon      │
│  (Krama)    │      │  (Vercel)    │      │  PostgreSQL │
└─────────────┘      └──────┬───────┘      └─────────────┘
                             │
                      ┌──────▼───────┐      ┌─────────────┐
                      │  Upstash     │      │  Telegram   │
                      │  Redis       │      │  (Storage)  │
                      └──────────────┘      └─────────────┘
```

## Prerequisites

Before deploying PixEdge, you need:

| Service | Purpose | Cost | Signup |
|---------|---------|------|--------|
| **Vercel Account** | Hosting | Free | https://vercel.com |
| **Neon PostgreSQL** | User accounts, sessions | Free tier (0.5GB) | https://console.neon.tech |
| **Upstash Redis** | Media metadata, analytics | Free tier (10K commands/day) | https://console.upstash.com |
| **Telegram Bot** | Storage via Bot API | Free | @BotFather |
| **Telegram Account** | Storage via MTProto | Free | https://telegram.org |

### Optional for 2GB Uploads (MTProto)

| Service | Purpose | Cost | Signup |
|---------|---------|------|--------|
| **Telegram API Credentials** | MTProto access | Free | https://my.telegram.org/apps |

---

## Step 1: Create Telegram Bot and Storage Channel

### 1.1 Create a Telegram Bot

1. Open Telegram and chat with [@BotFather](https://t.me/botfather)
2. Send `/newbot`
3. Follow prompts:
   - Bot name: `PixEdge_Bot` (or your choice)
   - Bot username: `your_pixedge_bot` (must end in `bot`)
4. Copy the bot token (format: `123456789:ABCdefGhIJKlmNoPQRsTUVwxyZ`)

### 1.2 Create Storage Channel

1. Create a new private channel in Telegram
2. Add your bot as **admin** with these permissions:
   - Post messages
   - Edit messages
   - Delete messages
3. Get the channel ID:
   - Method 1: Forward a message from the channel to [@userinfobot](https://t.me/userinfobot)
   - Method 2: Use API:
     ```
     https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
     ```
   - Channel IDs are typically negative (e.g., `-1001234567890`)

### 1.3 Create Log Channel (Optional)

Create another private channel for upload logs (or use the same as storage channel).

---

## Step 2: Set Up Neon PostgreSQL

### 2.1 Create Neon Project

1. Go to [https://console.neon.tech](https://console.neon.tech)
2. Click **New Project**
3. Configure:
   - Project name: `pixedge` (or your choice)
   - Region: Choose closest to your users
   - Database: `pixedge`
   - Username: `pixedge_user` (or your choice)
   - Password: **Copy this password** (you won't see it again)
4. Click **Create Project**

### 2.2 Get Connection String

From the Neon dashboard, copy the connection string:
```
postgresql://username:password@host/database?sslmode=require
```

### 2.3 Initialize Database Schema

The schema is automatically managed by Prisma. When you first deploy, run:

```bash
# Local development
npm run db:push

# This will create all necessary tables in Neon PostgreSQL
```

---

## Step 3: Set Up Upstash Redis

### 3.1 Create Redis Database

1. Go to [https://console.upstash.com](https://console.upstash.com)
2. Click **Create Database**
3. Configure:
   - Database name: `pixedge`
   - Region: Choose closest to your Vercel deployment
   - Database type: **Serverless** (pay-as-you-go for free tier)
4. Click **Create**

### 3.2 Copy Credentials

From the database dashboard, copy:
- **REST URL**: `https://your-redis.upstash.io`
- **REST Token**: `your_redis_token`

---

## Step 4: Deploy to Vercel

### 4.1 One-Click Deploy (Recommended)

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Forg-calm-moon-46812842%2FPixEdge)

### 4.2 Manual Deploy via GitHub

1. Fork the PixEdge repository to your GitHub account
2. Go to [https://vercel.com/new](https://vercel.com/new)
3. Click **Import Git Repository**
4. Select your forked PixEdge repo
5. Configure project:
   - Framework Preset: **Next.js**
   - Root Directory: `./` (or `pixedge`)
6. Click **Deploy**

### 4.3 Configure Environment Variables

After deployment, go to **Project Settings → Environment Variables** and add:

| Variable | Value | Description |
|----------|-------|-------------|
| `NEXT_PUBLIC_BASE_URL` | `https://your-domain.vercel.app` | Your Vercel deployment URL |
| `NEXTAUTH_URL` | `https://your-domain.vercel.app` | Same as BASE_URL |
| `NEXTAUTH_SECRET` | Random 32+ character string | Generate with: `openssl rand -base64 32` |
| `DATABASE_URL` | Neon connection string | `postgresql://...` |
| `TELEGRAM_BOT_TOKEN` | Your bot token | From @BotFather |
| `TELEGRAM_CHAT_ID` | Your storage channel ID | `-1001234567890` |
| `TELEGRAM_LOG_CHANNEL_ID` | Your log channel ID | Same as CHAT_ID or separate |
| `UPSTASH_REDIS_REST_URL` | Redis REST URL | `https://...` |
| `UPSTASH_REDIS_REST_TOKEN` | Redis REST Token | Token string |
| `GOOGLE_CLIENT_ID` | (Optional) | For Google OAuth |
| `GOOGLE_CLIENT_SECRET` | (Optional) | For Google OAuth |
| `GITHUB_ID` | (Optional) | For GitHub OAuth |
| `GITHUB_SECRET` | (Optional) | For GitHub OAuth |

### 4.4 Redeploy

After adding environment variables, go to **Deployments** and click **Redeploy** on the latest deployment.

---

## Step 5: Configure Telegram Webhook

After your PixEdge instance is live, configure the Telegram webhook:

```bash
# Replace YOUR_BOT_TOKEN and YOUR_DOMAIN
curl "https://api.telegram.org/bot<YOUR_BOT_TOKEN>/setWebhook?url=https://your-domain.vercel.app/api/webhook/telegram"
```

Expected response:
```json
{"ok":true,"result":true,"description":"Webhook was set"}
```

### Verify Webhook

```bash
curl "https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getWebhookInfo"
```

---

## Step 6: Set Up MTProto Session (Optional - for 2GB Uploads)

MTProto enables uploads up to 2GB instead of the Bot API's 20MB limit.

### 6.1 Get Telegram API Credentials

1. Go to [https://my.telegram.org/apps](https://my.telegram.org/apps)
2. Log in with your Telegram account
3. Click **Create application**
4. Fill in:
   - App title: `PixEdge`
   - Short name: `pixedge`
   - URL: `https://your-domain.vercel.app`
   - Platform: **Web**
5. Click **Create application**
6. Copy **App api_id** and **App api_hash**

### 6.2 Generate Session String

The session string authenticates your Telegram account for uploads.

```bash
# Clone and set up the project locally
git clone https://github.com/org-calm-moon-46812842/PixEdge.git
cd PixEdge
npm install

# Copy environment file
cp .env.example .env.local
# Edit .env.local and add:
#   - TELEGRAM_API_ID=your_api_id
#   - TELEGRAM_API_HASH=your_api_hash
#   - TELEGRAM_BOT_TOKEN=your_bot_token

# Generate the session
npm run generate:session
```

Follow the prompts:
1. Enter your phone number (format: `+1234567890`)
2. Enter the OTP code sent to your Telegram app
3. If prompted, enter your 2FA password
4. Copy the printed `TELEGRAM_SESSION_STRING`

### 6.3 Add MTProto Variables to Vercel

In Vercel project settings, add:

| Variable | Value |
|----------|-------|
| `TELEGRAM_API_ID` | Your API ID |
| `TELEGRAM_API_HASH` | Your API hash |
| `TELEGRAM_SESSION_STRING` | The generated session string |
| `MAX_UPLOAD_SIZE_MB` | `2000` (or lower if needed) |

### 6.4 Redeploy

Redeploy your project to activate MTProto mode.

---

## Step 7: Configure OAuth Providers (Optional)

### Google OAuth

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create credentials → **OAuth 2.0 Client ID**
3. Application type: **Web application**
4. Add authorized redirect URI:
   ```
   https://your-domain.vercel.app/api/auth/callback/google
   ```
5. Copy **Client ID** and **Client Secret**

### GitHub OAuth

1. Go to [GitHub Developer Settings](https://github.com/settings/developers)
2. **New OAuth App**
3. Fill in:
   - Application name: `PixEdge`
   - Homepage URL: `https://your-domain.vercel.app`
   - Callback URL: `https://your-domain.vercel.app/api/auth/callback/github`
4. Click **Register application**
5. Copy **Client ID** and **Client Secret**

---

## Step 8: Verify Deployment

### 8.1 Check Basic Functionality

1. Visit your deployed URL (e.g., `https://pixedge.vercel.app`)
2. You should see the PixEdge homepage
3. Try signing up with email/password

### 8.2 Test Media Upload

1. Log in to your PixEdge account
2. Go to Dashboard
3. Try uploading a test image
4. Verify the image appears in your Telegram storage channel
5. Copy the returned URL and verify it loads

### 8.3 Test API Upload

```bash
# Get your API key from Dashboard → API Keys
curl -X POST https://your-domain.vercel.app/api/v1/upload \
  -H "X-API-Key: your_api_key" \
  -F "file=@test-image.png"
```

Expected response:
```json
{
  "success": true,
  "id": "abc123",
  "url": "https://t.me/...",
  "metadata": {
    "filename": "test-image.png",
    "size": 12345,
    "mimeType": "image/png"
  }
}
```

### 8.4 Test Telegram Bot

1. Open Telegram and search for your bot
2. Send `/start`
3. Send `/help`
4. Try uploading an image directly to the bot

---

## Troubleshooting

### "Database connection failed"

1. Verify Neon DATABASE_URL is correct
2. Check SSL mode is enabled: `?sslmode=require`
3. Ensure Neon project is active (not paused)
4. Test connection with `psql`:
   ```bash
   psql "postgresql://user:pass@host/db?sslmode=require"
   ```

### "Telegram upload failed"

1. Verify bot token is correct
2. Ensure bot is admin in storage channel
3. Check channel ID format (must be negative for private channels)
4. Test bot manually:
   ```
   https://api.telegram.org/bot<TOKEN>/getMe
   ```

### "Redis connection failed"

1. Verify Upstash URL and token are correct
2. Check if Redis instance is active
3. Verify no IP restrictions in Upstash dashboard

### "Webhook not working"

1. Verify webhook URL is correct and accessible
2. Check Vercel logs for errors
3. Ensure HTTPS is enabled (required by Telegram)
4. Test manually:
   ```bash
   curl -v https://your-domain.vercel.app/api/webhook/telegram
   ```

### "MTProto uploads failing"

1. Verify TELEGRAM_API_ID and TELEGRAM_API_HASH are correct
2. Ensure TELEGRAM_SESSION_STRING is still valid
3. Check if session has expired (regenerate if needed)
4. Verify phone number has access to Telegram

### "OAuth not working"

1. Verify callback URLs match exactly
2. Check client ID/secret are correct
3. Ensure authorized domains are configured in OAuth apps
4. Check Vercel logs for specific OAuth errors

---

## Cost Analysis

### Free Tier Breakdown

| Service | Free Tier | Cost |
|---------|-----------|------|
| **Vercel** | 100GB bandwidth, 100K builds/min | $0/month |
| **Neon PostgreSQL** | 0.5GB storage | $0/month |
| **Upstash Redis** | 10K commands/day | $0/month |
| **Telegram** | Unlimited storage | $0/month |
| **GitHub OAuth** | Unlimited | $0/month |
| **Google OAuth** | Unlimited | $0/month |

### Total Monthly Cost: $0

---

## Production Checklist

Before going live:

- [ ] All environment variables configured in Vercel
- [ ] Telegram webhook verified
- [ ] Database schema initialized
- [ ] OAuth providers configured (if using social login)
- [ ] MTProto session generated (if using 2GB uploads)
- [ ] Custom domain configured in Vercel (optional)
- [ ] SSL certificate active (automatic with Vercel)
- [ ] Test uploads working
- [ ] Bot commands registered with @BotFather
- [ ] PixEdge API key generated for Krama integration

---

## Integration with Krama

After deployment, configure Krama to use PixEdge:

1. **Get API Key**: Dashboard → API Keys → Generate new key
2. **Update Krama .env**:
   ```
   PIXEDGE_API_URL=https://your-pixedge.vercel.app
   PIXEDGE_API_KEY=pe_your_api_key_here
   ```
3. **Test Integration**: Large files (>10MB) will automatically use PixEdge

See [API_INTEGRATION.md](./API_INTEGRATION.md) for detailed API documentation.

---

## Support

- **GitHub Issues**: https://github.com/org-calm-moon-46812842/PixEdge/issues
- **Telegram Channel**: [@EdgeBots](https://t.me/EdgeBots)
- **Support Chat**: [@EdgeBotSupport](https://t.me/EdgeBotSupport)
