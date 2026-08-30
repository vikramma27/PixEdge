# PixEdge API Integration Guide

API documentation for integrating PixEdge media storage with Krama messaging app.

## Overview

PixEdge provides a RESTful API for uploading and managing media files. The Krama app uses PixEdge as a fallback for large media files (>10MB) when Matrix upload fails.

## Base URL

```
Production: https://your-pixedge-domain.vercel.app
```

## Authentication

All authenticated endpoints accept authentication via:

### Option 1: API Key Header (Recommended)
```bash
X-API-Key: pe_your_api_key_here
```

### Option 2: Bearer Token
```bash
Authorization: Bearer pe_your_api_key_here
```

### Option 3: Session Cookie
For browser-based uploads, the session cookie from logging in is automatically used.

---

## API Endpoints

### Upload Media

**`POST /api/v1/upload`**

Upload a media file to PixEdge.

**Request:**

```
Content-Type: multipart/form-data
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | File | Yes | Image, video, or any file |
| `customId` | String | No | Custom vanity slug (unique identifier) |
| `expiresIn` | Number | No | Seconds until expiry: `3600`, `86400`, `604800`, `2592000` |

**Example:**
```bash
curl -X POST https://your-pixedge-domain.vercel.app/api/v1/upload \
  -H "X-API-Key: pe_your_api_key" \
  -F "file=@video.mp4" \
  -F "customId=my-video" \
  -F "expiresIn=86400"
```

**Success Response:**
```json
{
  "success": true,
  "id": "abc123def456",
  "url": "https://t.me/pixedge_bot/photo/1",
  "metadata": {
    "filename": "video.mp4",
    "size": 15728640,
    "mimeType": "video/mp4",
    "width": 1920,
    "height": 1080,
    "duration": 120
  },
  "expiresAt": "2024-02-01T12:00:00Z",
  "createdAt": "2024-01-31T12:00:00Z"
}
```

**Error Responses:**

| Status | Error | Description |
|--------|-------|-------------|
| 400 | `FILE_TOO_LARGE` | File exceeds size limit |
| 400 | `INVALID_FILE_TYPE` | File type not allowed |
| 401 | `UNAUTHORIZED` | Invalid or missing API key |
| 429 | `RATE_LIMITED` | Too many uploads |
| 500 | `UPLOAD_FAILED` | Telegram upload error |

### Get File Info

**`GET /api/v1/info/[id]`**

Get metadata for an uploaded file.

**Example:**
```bash
curl https://your-pixedge-domain.vercel.app/api/v1/info/abc123def456 \
  -H "X-API-Key: pe_your_api_key"
```

**Response:**
```json
{
  "success": true,
  "id": "abc123def456",
  "url": "https://t.me/pixedge_bot/photo/1",
  "metadata": {
    "filename": "video.mp4",
    "size": 15728640,
    "mimeType": "video/mp4"
  },
  "views": 42,
  "downloads": 12,
  "createdAt": "2024-01-31T12:00:00Z",
  "expiresAt": "2024-02-01T12:00:00Z"
}
```

### List User Uploads

**`GET /api/v1/list`**

List all uploads for the authenticated user.

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | Number | 50 | Max results (1-100) |
| `offset` | Number | 0 | Pagination offset |

**Example:**
```bash
curl "https://your-pixedge-domain.vercel.app/api/v1/list?limit=20&offset=0" \
  -H "X-API-Key: pe_your_api_key"
```

**Response:**
```json
{
  "success": true,
  "uploads": [
    {
      "id": "abc123def456",
      "url": "https://t.me/pixedge_bot/photo/1",
      "metadata": {
        "filename": "video.mp4",
        "size": 15728640,
        "mimeType": "video/mp4"
      },
      "views": 42,
      "downloads": 12,
      "createdAt": "2024-01-31T12:00:00Z",
      "expiresAt": "2024-02-01T12:00:00Z"
    }
  ],
  "total": 150,
  "limit": 20,
  "offset": 0
}
```

### Delete File

**`DELETE /api/v1/delete/[id]`**

Delete an uploaded file.

**Example:**
```bash
curl -X DELETE https://your-pixedge-domain.vercel.app/api/v1/delete/abc123def456 \
  -H "X-API-Key: pe_your_api_key"
```

**Response:**
```json
{
  "success": true,
  "message": "File deleted successfully"
}
```

### Get Platform Stats

**`GET /api/stats`**

Public endpoint for platform statistics. No authentication required.

**Response:**
```json
{
  "success": true,
  "stats": {
    "totalUploads": 15000,
    "totalStorage": "500GB",
    "totalBandwidth": "100TB",
    "activeUsers": 500
  }
}
```

### Telegram Bot Webhook

**`POST /api/webhook/telegram`**

Receives updates from Telegram bot. No authentication (uses bot token verification).

---

## Krama Integration Flow

### Media Upload Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Krama Android App                            │
├─────────────────────────────────────────────────────────────────────┤
│  1. User selects media to send                                      │
│  2. App tries Matrix upload first (E2E encrypted)                   │
│  3. If Matrix fails OR file > 10MB:                                 │
│     └──▶ 4. App calls PixEdge API                                   │
│              POST /api/v1/upload                                    │
│              X-API-Key: pe_krama_api_key                            │
│              file: <media_file>                                     │
│  5. On success:                                                     │
│     └──▶ 6. Get PixEdge URL                                         │
│  7. Store PixEdge URL in Matrix message as fallback                  │
│  8. Recipient downloads media from PixEdge URL                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Upload Sequence Diagram

```
Krama                    Matrix                 PixEdge              Telegram
  │                        │                       │                     │
  │  1. Select Media       │                       │                     │
  │───────────────────────▶│                       │                     │
  │                        │                       │                     │
  │  2. Try Matrix Upload  │                       │                     │
  │                        │───────────────────────▶│                     │
  │                        │   (if >10MB or fails) │                     │
  │                        │                       │                     │
  │                        │                       │  3. Upload via Bot  │
  │                        │                       │─────────────────────▶│
  │                        │                       │                     │
  │                        │                       │  4. Store in Channel│
  │                        │                       │◀─────────────────────│
  │                        │                       │                     │
  │                        │  5. Return PixEdge URL                     │
  │◀───────────────────────│                       │                     │
  │                        │                       │                     │
  │  6. Send Matrix msg   │                       │                     │
  │  with PixEdge URL      │                       │                     │
  │                        │                       │                     │
```

### Error Handling

| Error Code | Meaning | Krama Action |
|------------|---------|--------------|
| `FILE_TOO_LARGE` | Exceeds 2GB limit | Show error to user |
| `INVALID_FILE_TYPE` | Unsupported type | Show error to user |
| `RATE_LIMITED` | Too many requests | Retry with backoff |
| `UPLOAD_FAILED` | Telegram error | Retry or show error |
| `UNAUTHORIZED` | Invalid API key | Check configuration |

### Retry Strategy

```kotlin
suspend fun uploadWithRetry(
    file: File,
    maxRetries: Int = 3,
    initialDelayMs: Long = 1000
): Result<PixEdgeResponse> {
    var delay = initialDelayMs
    repeat(maxRetries) { attempt ->
        val result = pixEdgeApi.upload(file)
        if (result.isSuccess) return result
        if (result.error == "RATE_LIMITED") {
            delay(delay)
            delay *= 2 // Exponential backoff
        } else {
            return result
        }
    }
    return Result.failure(Exception("Upload failed after $maxRetries retries"))
}
```

---

## Rate Limits

| Endpoint | Authenticated | Anonymous |
|----------|--------------|-----------|
| `POST /api/v1/upload` | 100/min | 20/min |
| `GET /api/v1/list` | 100/min | 20/min |
| `GET /api/v1/info/[id]` | Unlimited | Unlimited |
| `DELETE /api/v1/delete/[id]` | 50/min | N/A |

Rate limit headers:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1706745600
```

---

## File Size Limits

| Mode | Max Size |
|------|----------|
| Bot API (default) | 20 MB |
| MTProto (configured) | 2 GB |

### Content Types

Allowed MIME types:
- Images: `image/jpeg`, `image/png`, `image/gif`, `image/webp`
- Videos: `video/mp4`, `video/webm`, `video/quicktime`
- Audio: `audio/mpeg`, `audio/ogg`, `audio/wav`
- Documents: `application/pdf`, `application/zip`

---

## Example Integration Code

### Kotlin (Krama Android App)

```kotlin
data class PixEdgeConfig(
    val apiUrl: String,
    val apiKey: String
)

data class PixEdgeResponse(
    val success: Boolean,
    val id: String?,
    val url: String?,
    val metadata: MediaMetadata?,
    val error: String?
)

data class MediaMetadata(
    val filename: String,
    val size: Long,
    val mimeType: String
)

class PixEdgeClient(private val config: PixEdgeConfig) {

    suspend fun uploadMedia(file: File): Result<PixEdgeResponse> {
        return try {
            val client = OkHttpClient()
            val mediaType = MediaType.parse("multipart/form-data")

            val requestBody = MultipartBody.Builder()
                .setType(MultipartBody.FORM)
                .addFormDataPart(
                    "file",
                    file.name,
                    RequestBody.create(mediaType, file)
                )
                .build()

            val request = Request.Builder()
                .url("${config.apiUrl}/api/v1/upload")
                .addHeader("X-API-Key", config.apiKey)
                .post(requestBody)
                .build()

            val response = client.newCall(request).execute()
            val body = response.body?.string()

            if (response.isSuccessful) {
                val pixResponse = parseResponse(body)
                if (pixResponse?.success == true) {
                    Result.success(pixResponse)
                } else {
                    Result.failure(Exception(pixResponse?.error ?: "Upload failed"))
                }
            } else {
                Result.failure(Exception("HTTP ${response.code}: Upload failed"))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }

    private fun parseResponse(body: String?): PixEdgeResponse? {
        // Parse JSON response
        return null // Implementation depends on JSON library
    }
}
```

### Usage in Krama

```kotlin
class MediaUploadEngine(
    private val matrixClient: MatrixClient,
    private val pixEdgeClient: PixEdgeClient
) {
    companion object {
        private const val MAX_MATRIX_SIZE = 10 * 1024 * 1024 // 10MB
    }

    suspend fun uploadMedia(file: File): Result<String> {
        // Try Matrix first for small files
        if (file.size <= MAX_MATRIX_SIZE) {
            val matrixResult = matrixClient.uploadMedia(file)
            if (matrixResult.isSuccess) {
                return Result.success(matrixResult.getOrThrow())
            }
        }

        // Fallback to PixEdge for large files or Matrix failure
        val pixEdgeResult = pixEdgeClient.uploadMedia(file)
        if (pixEdgeResult.isSuccess) {
            return Result.success(pixEdgeResult.getOrThrow().url!!)
        }

        return Result.failure(Exception("All upload methods failed"))
    }
}
```

---

## Security Considerations

1. **API Key Protection**: Never expose API keys in client-side code
2. **HTTPS Only**: All API calls must use HTTPS
3. **Rate Limiting**: Implement client-side rate limiting
4. **File Validation**: Validate file types and sizes before upload
5. **Error Messages**: Don't expose internal errors to users

---

## Troubleshooting

### "Upload failed" in Krama

1. Verify PixEdge URL and API key are correct
2. Check internet connection
3. Try uploading directly via curl to test
4. Check Vercel deployment logs

### Files not appearing in Telegram

1. Verify bot is admin in storage channel
2. Check channel ID format is correct
3. Test bot manually in Telegram

### Rate limit errors

1. Implement exponential backoff
2. Batch uploads when possible
3. Consider multiple API keys for high volume
