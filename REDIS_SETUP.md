# Redis Setup for Transcript Caching

## Overview
This project uses Redis to cache YouTube transcripts, avoiding redundant API calls to Supadata for the same video.

## Configuration

### Environment Variables
Add to `.env.local`:
```env
REDIS_URL=redis://localhost:6379
```

If you're using a Redis instance with authentication:
```env
REDIS_URL=redis://:password@localhost:6379
```

For different host/port:
```env
REDIS_URL=redis://your-host:your-port
```

## Local Redis Setup

### Option 1: Using Docker (Recommended)
```bash
docker run -d -p 6379:6379 redis:latest
```

### Option 2: Install Redis Locally
**macOS (Homebrew):**
```bash
brew install redis
brew services start redis
```

**Windows (WSL2):**
```bash
# Inside WSL2
sudo apt-get install redis-server
redis-server
```

**Ubuntu/Debian:**
```bash
sudo apt-get install redis-server
sudo systemctl start redis-server
```

## Verify Redis Connection
```bash
# Connect to Redis CLI
redis-cli

# Check if connected
127.0.0.1:6379> ping
PONG
```

## Cache Strategy

### Cache Key Format
- **Pattern**: `transcript:{videoId}`
- **Example**: `transcript:dQw4w9WgXcQ`

### Expiration Policy
- **TTL**: 7 days (604,800 seconds)
- **Auto-expiration**: Redis automatically deletes expired keys

### How It Works
1. **First Request**:
   - Cache miss → Fetch from Supadata
   - Store result in Redis for 7 days
   - Return to user

2. **Subsequent Requests** (within 7 days):
   - Cache hit → Return immediately from Redis
   - ~1000ms faster than API call
   - No Supadata quota used

## Monitoring Cache

### Check Cache Stats
```bash
redis-cli
> INFO stats
> DBSIZE  # Total keys in Redis
> KEYS transcript:*  # List all cached transcripts
```

### Manually Clear Cache
```bash
redis-cli
> FLUSHDB  # Clear all keys in current database
> DEL transcript:dQw4w9WgXcQ  # Delete specific video
```

## Logging

When transcript caching is working:
- **Cache Hit**: `Cache hit for video {videoId}, length: {length}`
- **Cache Miss**: `Cache miss for video {videoId}, fetching from Supadata`
- **Cache Stored**: `Cached transcript for video {videoId}`

## Troubleshooting

### Connection Error: "Error: connect ECONNREFUSED 127.0.0.1:6379"
- Redis is not running
- **Fix**: Start Redis with `redis-server` or Docker command above

### Slow Performance
- Redis may be overloaded
- **Fix**: Run `redis-cli FLUSHDB` to clear cache and start fresh

### Cache Not Working
- Check Redis is connected: `redis-cli ping` should return `PONG`
- Verify `REDIS_URL` in `.env.local` is correct
- Check browser console and server logs for errors
