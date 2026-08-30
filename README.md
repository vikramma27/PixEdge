# PixEdge - Ultra-Fast Edge Media Hosting

**PixEdge** is a professional-grade, open-source media hosting platform that uses Telegram as a free, unlimited storage backend. Deploy to Vercel in minutes with zero storage costs.

[![Version](https://img.shields.io/badge/version-2.0.0-blue?style=for-the-badge)](https://github.com/org-calm-moon-46812842/PixEdge)
[![Next.js](https://img.shields.io/badge/Next.js-15-black?style=for-the-badge&logo=nextdotjs&logoColor=white)](https://nextjs.org/)
[![Neon](https://img.shields.io/badge/Neon-00E699?style=for-the-badge&logo=neon&logoColor=black)](https://neon.tech)
[![Telegram](https://img.shields.io/badge/Telegram-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)](https://telegram.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

---

## Key Features

| Feature | Description |
|---------|-------------|
| **Unlimited Storage** | Powered by Telegram infrastructure |
| **2 GB Uploads** | MTProto mode breaks the 20 MB Bot API cap |
| **Edge Delivery** | 302 redirect to Telegram CDN |
| **True Streaming** | v2 files streamed in 1 MB chunks |
| **User Accounts** | Email, GitHub, Google, Telegram OAuth |
| **API Access** | Programmatic uploads via REST API |
| **ShareX Ready** | One-click `.sxcu` configuration |
| **Expiry Links** | Auto-expire links after 1h/24h/7d/30d |

---

## Quick Start

### 1. Create Telegram Bot

1. Open [@BotFather](https://t.me/botfather) on Telegram
2. Send `/newbot` and follow the prompts
3. Copy the bot token

### 2. Create Storage Channel

1. Create a private channel
2. Add your bot as **admin** with posting permissions
3. Get the channel ID (forward a message to [@userinfobot](https://t.me/userinfobot))

### 3. Set Up Services

| Service | Free Tier | Signup |
|---------|-----------|--------|
| **Vercel** | 100GB bandwidth | [vercel.com](https://vercel.com) |
| **Neon PostgreSQL** | 0.5GB storage | [console.neon.tech](https://console.neon.tech) |
| **Upstash Redis** | 10K commands/day | [console.upstash.com](https://console.upstash.com) |

### 4. Deploy to Vercel

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Forg-calm-moon-46812842%2FPixEdge)

### 5. Configure Environment Variables

In Vercel project settings, add:

```env
NEXT_PUBLIC_BASE_URL=https://your-domain.vercel.app
NEXTAUTH_SECRET=your_32_char_random_secret
NEXTAUTH_URL=https://your-domain.vercel.app
DATABASE_URL=postgresql://user:pass@host/db?sslmode=require
TELEGRAM_BOT_TOKEN=your_bot_token
TELEGRAM_CHAT_ID=-1001234567890
UPSTASH_REDIS_REST_URL=https://your-redis.upstash.io
UPLEASH_REDIS_REST_TOKEN=your_token
```

### 6. Set Telegram Webhook

```bash
curl "https://api.telegram.org/bot<YOUR_BOT_TOKEN>/setWebhook?url=https://your-domain.vercel.app/api/webhook/telegram"
```

---

## API Reference

### Upload Media

```bash
curl -X POST https://your-domain.com/api/v1/upload \
  -H "X-API-Key: your_api_key" \
  -F "file=@video.mp4"
```

### Response

```json
{
  "success": true,
  "id": "abc123",
  "url": "https://t.me/...",
  "metadata": {
    "filename": "video.mp4",
    "size": 15728640,
    "mimeType": "video/mp4"
  }
}
```

### API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/upload` | Upload a file |
| `GET` | `/api/v1/info/[id]` | Get file metadata |
| `GET` | `/api/v1/list` | List user uploads |
| `DELETE` | `/api/v1/delete/[id]` | Delete a file |
| `GET` | `/api/stats` | Public platform stats |

---

## MTProto Setup (Optional)

For 2 GB uploads instead of 20 MB:

1. Get credentials at [my.telegram.org/apps](https://my.telegram.org/apps)
2. Generate session:
   ```bash
   npm run generate:session
   ```
3. Add to environment:
   ```env
   TELEGRAM_API_ID=your_api_id
   TELEGRAM_API_HASH=your_api_hash
   TELEGRAM_SESSION_STRING=your_session_string
   ```

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Framework | Next.js 15 (App Router) |
| Auth | NextAuth.js |
| Database | Neon PostgreSQL |
| Cache | Upstash Redis |
| Storage | Telegram Bot API / MTProto |
| Styling | Vanilla CSS + Framer Motion |

---

## Architecture

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│  Client  │────▶│ Next.js  │────▶│  Neon    │
│          │     │  (Vercel)│     │ PostgreSQL│
└──────────┘     └────┬─────┘     └──────────┘
                      │
              ┌───────┼───────┐
              │               │
        ┌─────▼─────┐   ┌────▼────┐
        │  Upstash  │   │ Telegram │
        │   Redis   │   │ (Storage)│
        └───────────┘   └─────────┘
```

---

## Cost Analysis

| Service | Free Tier | Notes |
|---------|-----------|-------|
| Vercel | 100GB/mo | Sufficient for ~100 users |
| Neon PostgreSQL | 0.5GB | Stores user accounts |
| Upstash Redis | 10K cmd/day | Stores metadata |
| Telegram | Unlimited | Free storage |

**Total: $0/month**

---

## File Structure

```
PixEdge/
├── prisma/
│   └── schema.prisma       # Database schema
├── src/
│   ├── app/
│   │   ├── api/           # API routes
│   │   │   ├── auth/      # NextAuth endpoints
│   │   │   ├── upload/    # Upload handling
│   │   │   └── webhook/   # Telegram webhook
│   │   ├── dashboard/     # User dashboard
│   │   └── page.tsx       # Landing page
│   └── lib/
│       ├── auth.ts         # NextAuth config
│       ├── db.ts          # Redis operations
│       └── telegram.ts     # Bot API wrapper
├── scripts/
│   └── generate-session.mjs  # MTProto session generator
├── .env.example           # Environment template
├── vercel.json            # Vercel config
└── package.json
```

---

## License

MIT License - see [LICENSE](LICENSE) file for details.
