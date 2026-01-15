# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**TLDW YT Video** is a Next.js web application that generates AI-powered summaries of YouTube videos. Users paste a video URL, select a summary length (brief, detailed, or comprehensive), and receive an instant summary powered by OpenAI's GPT-4o-mini API.

**Key Characteristics:**
- Single-page Next.js 15 App Router application
- Minimal backend (one API route for summarization)
- Real-time video transcript fetching via Supadata API
- Redis-powered transcript caching (7-day TTL)
- Adaptive time estimation based on historical performance
- Real-time URL validation with helpful error messages
- Visual feedback for user actions (copy to clipboard, loading progress)
- TypeScript throughout with strict mode enabled
- Tailwind CSS for responsive, modern styling
- pnpm as the package manager

## Architecture & Code Organization

### Core Structure
```
app/
├── layout.tsx                 # Root layout with metadata and global config
├── page.tsx                   # Main UI component (client-side, use client)
├── globals.css                # Tailwind imports and global styles
├── components/
│   └── LoadingState.tsx        # Loading overlay with progress indicators and time estimation
└── api/
    └── summarize/
        └── route.ts           # POST endpoint for video summarization with caching
lib/
├── redis.ts                   # Redis client initialization and connection pool
├── youtube-validation.ts      # URL validation and video ID extraction
├── timing-history.ts          # Historical timing data for adaptive estimation
└── supadata.types.ts          # TypeScript types for Supadata API responses
```

### Data Flow
1. **Client (page.tsx)**:
   - User enters YouTube URL with real-time validation
   - Selects summary length (brief/detailed/comprehensive)
   - Submits form to API endpoint
2. **API Route (route.ts)**:
   - Validates input (URL format, length type)
   - Extracts video ID from URL using regex patterns
   - Checks Redis cache for existing transcript (7-day TTL)
   - If cache miss: Fetches transcript from Supadata API
   - Stores transcript in Redis with TTL for future requests
   - Generates prompt with length-based instructions
   - Calls OpenAI GPT-4o-mini API with token limits: brief=500, detailed=1000, comprehensive=2000
   - Returns structured JSON response with summary and metadata
3. **Client Display**:
   - Shows summary with formatted text
   - Records timing data for adaptive estimation
   - Copy-to-clipboard button with visual feedback

### Key Implementation Details

**Video ID Extraction**: `route.ts:155-165`
- Supports: `youtube.com/watch?v=`, `youtu.be/`, `youtube.com/embed/`, `youtube.com/v/`
- Uses `extractVideoId()` utility function from `lib/youtube-validation.ts`
- Validates URL format before attempting extraction

**URL Validation**: `lib/youtube-validation.ts`
- Real-time validation during user input (page.tsx:31-42)
- Comprehensive error messages for different URL format issues
- Supported formats: standard watch URLs, short URLs, embed URLs, and more

**Transcript Fetching with Caching**: `route.ts:19-77`
- **Cache Layer** (19-30):
  - Checks Redis for cached transcript using key `transcript:{videoId}`
  - Returns cached transcript immediately if found (no API call needed)
  - Logs cache hit/miss for debugging
- **Supadata Integration** (32-67):
  - Fetches transcript from Supadata YouTube API
  - Requests plain text format (`text: true`) for cleaner output
  - Handles both string and TranscriptChunk[] response formats
  - Validates response has content before caching
- **Cache Storage** (64-65):
  - Stores in Redis with 7-day TTL (604,800 seconds)
  - Configured in `TRANSCRIPT_CACHE_TTL` constant

**Prompt Engineering**: `route.ts:80-97`
- Length-based instructions:
  - **Brief**: "2-3 paragraphs with the main points"
  - **Detailed**: "5-7 paragraphs covering the main topics and key points"
  - **Comprehensive**: "Full coverage with all major topics, key arguments, and conclusions"
- Structure: System context → length-specific instruction → transcript → request for summary

**Token Limits**: `route.ts:182`
- Brief: 500 tokens
- Detailed: 1000 tokens
- Comprehensive: 2000 tokens
- Configurable in `route.ts:182` ternary logic

**Loading State & Time Estimation**: `app/components/LoadingState.tsx`
- Shows progress indicators during API calls
- Retrieves historical timing data from `lib/timing-history.ts`
- Displays adaptive time estimates based on summary length
- Client-side estimation: distributes total time across phases (15% fetch, 15% process, rest generate)

**Timing History**: `lib/timing-history.ts`
- Records completion times for each summary length
- Stores in browser localStorage for persistence
- Calculates average times for adaptive estimation
- Used by LoadingState component to show realistic time expectations

## Development Commands

```bash
# Start development server (hot reload)
pnpm dev

# Build for production
pnpm build

# Start production server (requires prior build)
pnpm start

# TypeScript type checking
pnpm tsc --noEmit

# ESLint (extends Next.js core-web-vitals)
pnpm lint
```

## Environment Setup

**Required Environment Variables:**
- `OPENAI_API_KEY` - OpenAI API key for GPT-4o-mini model
- `SUPADATA_API_KEY` - Supadata API key for YouTube transcript fetching
- `REDIS_URL` - Redis connection URL (format: `redis://hostname:port` or `redis://:password@hostname:port`)

**Configuration Files:**
- `.env.local` - Local environment variables (not committed, git-ignored)
- `.env.example` - Template with required variables
- `tsconfig.json` - Path alias: `@/*` maps to project root
- `.eslintrc.json` - Extends Next.js core-web-vitals config
- `next.config.js` - Next.js configuration (minimal setup)
- `tailwind.config.ts` - Tailwind CSS configuration
- `postcss.config.js` - PostCSS configuration for Tailwind

## Common Development Tasks

### Adding a New Summary Length Option
1. Update the union type in `page.tsx:8` (SummaryLength type)
2. Add radio button option to the form (`page.tsx:160-182`)
3. Add corresponding instruction in `generatePrompt()` function (`route.ts:81-85` in lengthInstructions object)
4. Set token limit in the POST handler (`route.ts:182` ternary logic)

### Adjusting Summary Token Limits
Located in `route.ts:182`. Adjust the ternary logic to change max_tokens for each length category.

### Modifying the Prompt Template
Edit `generatePrompt()` function in `route.ts:80-97`. The structure is: system context → length-specific instruction → transcript → request for summary.

### UI/Styling Changes
- Main component: `page.tsx` (inline Tailwind classes, client component)
- Loading overlay: `app/components/LoadingState.tsx` (progress indicators and time estimation)
- Global styles: `app/globals.css` (Tailwind imports and base styles)
- Layout wrapper: `app/layout.tsx` (metadata and head configuration)

### Managing Redis Cache
- Cache TTL: 7 days (configured in `route.ts:17` TRANSCRIPT_CACHE_TTL)
- Cache key format: `transcript:{videoId}`
- To clear transcript cache: Delete Redis key or restart Redis
- Useful during development if Supadata responses change

### Testing the Transcript Fetch
The Supadata API requires:
1. Valid Supadata API key in `SUPADATA_API_KEY` environment variable
2. Valid video ID extracted from URL
3. Video must have public English captions available
4. Redis connection for caching

If transcript fetch fails, check:
- `route.ts:68-76` error handling and logging
- Supadata API response format and content
- Redis connection status (`REDIS_URL` configuration)
- Video caption availability on YouTube
- API key quota and permissions

### Working with Redis
Local development setup:
```bash
# Start Redis locally (requires Redis installed)
redis-server

# Or use Docker
docker run -d -p 6379:6379 redis:latest

# Connect via CLI to inspect cache
redis-cli
> KEYS transcript:*  # View all cached transcripts
> TTL transcript:VIDEO_ID  # Check remaining TTL
> DEL transcript:VIDEO_ID  # Remove specific transcript
```

## API Endpoint Reference

**POST /api/summarize**

Request:
```json
{
  "url": "https://www.youtube.com/watch?v=...",
  "length": "detailed"
}
```

Response (Success):
```json
{
  "success": true,
  "videoId": "dQw4w9WgXcQ",
  "summary": "...",
  "length": "detailed"
}
```

Response (Error):
```json
{
  "error": "Error description"
}
```

Status Codes:
- 200: Success
- 400: Invalid input or missing transcript
- 500: API/processing error

## Important Notes

**API Keys**:
- Store `OPENAI_API_KEY`, `SUPADATA_API_KEY`, and `REDIS_URL` in `.env.local` only—never commit to git
- `.gitignore` already configured to exclude `.env.local`
- Each API has quota limits—monitor usage in respective dashboards

**Redis Configuration**:
- Required for production and local development (transcript caching)
- Connection failures will prevent transcript fetching
- Consider Redis Atlas or similar managed services for production
- Local development: `redis://localhost:6379` (default)
- Production: Use connection pooling and SSL/TLS encryption

**YouTube Transcript & Supadata Limitations**:
- Only works with videos that have public English captions
- Some videos have captions disabled by uploader
- Very long videos may have truncated transcripts due to API limits
- Supadata's transcription quality depends on YouTube's caption availability
- Caching reduces API calls but requires memory management

**Error Handling**:
- Frontend catches fetch errors and displays user-friendly messages
- API route logs detailed errors to console for debugging
- Invalid URLs validated both client-side and server-side
- Redis connection failures gracefully reported to client

**TypeScript Strict Mode**:
- Enabled in `tsconfig.json`
- All code must pass strict type checking
- Use `pnpm tsc --noEmit` to validate types before commit

**Performance Considerations**:
- Transcript caching (7-day TTL) significantly reduces API calls
- Redis cache hit: ~100-200ms response time
- Supadata API call: ~2-5 seconds per transcript
- OpenAI API call: ~3-10 seconds depending on summary length
- Total end-to-end: ~5-15 seconds (first request), ~3-7 seconds (cached)

## Multi-Language Support

**NEW FEATURE**: YouTube TLDR now supports transcripts and summaries in 19 different languages.

**Key Capabilities**:
- Auto-detect transcript language from YouTube captions
- Manual language selection for specific caption tracks
- Generate summaries in any supported language (English, Spanish, French, German, etc.)
- Cross-language translation (e.g., Spanish video → English summary)
- Language-specific caching for improved performance

**Files Related to Multi-Language**:
- `lib/language-utils.ts` - Language definitions and utilities
- `app/page.tsx` - Language selection UI components
- `app/api/summarize/route.ts` - Language-aware transcript fetching and summarization
- `docs/MULTI_LANGUAGE.md` - Complete multi-language documentation

**See**: `docs/MULTI_LANGUAGE.md` for complete usage guide, API changes, and examples.

## File Purpose Reference

| File | Purpose |
|------|---------|
| `app/page.tsx` | Main UI form and summary display (React client component) with language selection |
| `app/api/summarize/route.ts` | POST endpoint handling transcript fetch, caching, and multi-language summarization |
| `app/components/LoadingState.tsx` | Loading overlay with progress indicators and adaptive time estimation |
| `app/layout.tsx` | Root HTML structure and metadata configuration |
| `app/globals.css` | Tailwind CSS imports and global styles |
| `lib/redis.ts` | Redis client initialization and connection management |
| `lib/youtube-validation.ts` | URL validation and video ID extraction utilities |
| `lib/timing-history.ts` | Timing history tracking for adaptive time estimation |
| `lib/language-utils.ts` | Language definitions, detection, and utility functions |
| `lib/supadata.types.ts` | TypeScript type definitions for Supadata API responses |
| `tsconfig.json` | TypeScript configuration with path alias for `@/` |
| `.eslintrc.json` | ESLint rules (extends Next.js core-web-vitals) |
| `next.config.js` | Next.js configuration (minimal) |
| `tailwind.config.ts` | Tailwind CSS configuration |
| `postcss.config.js` | PostCSS configuration (for Tailwind) |
| `package.json` | Dependencies: next, react, openai, @supadata/js, redis |
| `.env.example` | Template for environment variables (commit this, not `.env.local`) |
| `.env.local` | Local environment variables (git-ignored, not committed) |
| `docs/MULTI_LANGUAGE.md` | Multi-language feature documentation and usage guide |
