# TLDW YT Video - Video Summarizer

A modern web application that generates instant summaries of YouTube videos with customizable summary lengths, intelligent caching, and adaptive time estimation.

## Features

- 🎬 **Paste YouTube URL** → Get instant summary
- 📝 **Customizable Summary Length** - Choose between brief, detailed, or comprehensive summaries
- 🌍 **Multi-Language Support** - Transcripts and summaries in 19 languages with auto-detection
- 🤖 **AI-Powered** - Uses OpenAI GPT-4o-mini for intelligent summarization
- 🎨 **Clean UI** - Modern, responsive design with Tailwind CSS and visual feedback
- ⚡ **Fast Processing** - Quick turnaround time with intelligent caching and progress indicators
- 📋 **Copy to Clipboard** - Easily copy summaries for sharing with visual feedback
- 💾 **Smart Caching** - Redis-powered transcript caching (7-day TTL) for faster subsequent requests
- ⏱️ **Adaptive Time Estimation** - Loading state shows estimated completion time based on historical data

## Tech Stack

- **Frontend**: React 19, Next.js 15 (App Router), TypeScript
- **Styling**: Tailwind CSS
- **Backend**: Next.js API Routes
- **AI**: OpenAI GPT-4o-mini API
- **Transcript Source**: Supadata API (YouTube transcription service)
- **Caching**: Redis for transcript storage
- **Database**: Redis

## Prerequisites

- Node.js 18+ or pnpm 8+
- An OpenAI API key (get one at https://platform.openai.com/)
- A Supadata API key for YouTube transcription (get one at https://supadata.ai/)
- Redis server running (local or remote)

## Installation

1. **Clone/Setup the project**
   ```bash
   cd tldr-yt-video
   pnpm install
   ```

2. **Configure Environment Variables**
   - Copy `.env.example` to `.env.local`
   - Add your API keys and Redis configuration:
     ```
     OPENAI_API_KEY=your_openai_api_key
     SUPADATA_API_KEY=your_supadata_api_key
     REDIS_URL=redis://localhost:6379
     ```

## Running Locally

```bash
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

## How It Works

1. **User Input**: Paste a YouTube URL, select summary length, and choose languages (transcript & summary)
2. **URL Validation**: Real-time validation of YouTube URLs with helpful error messages
3. **Transcript Extraction**: The app fetches the YouTube video transcript using Supadata's API
4. **Language Detection**: Auto-detects transcript language or uses manual selection
5. **Smart Caching**: Transcripts are cached in Redis per video + language for 7 days
6. **AI Summarization**: OpenAI GPT-4o-mini processes and translates (if needed) the transcript
7. **Progress Feedback**: Loading state shows progress indicators and adaptive time estimation
8. **Display**: The summary is displayed with detected language badge and copy-to-clipboard functionality

### Supported Summary Lengths

- **Brief** - 2-3 paragraphs with main points (max 500 tokens)
- **Detailed** - 5-7 paragraphs covering main topics (max 1000 tokens)
- **Comprehensive** - Full coverage with all major topics and conclusions (max 2000 tokens)

## API Endpoint

### POST /api/summarize

Generates a summary for a YouTube video.

**Request Body:**
```json
{
  "url": "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
  "length": "detailed",
  "transcriptLanguage": "es",
  "summaryLanguage": "English"
}
```

**Response:**
```json
{
  "success": true,
  "videoId": "dQw4w9WgXcQ",
  "summary": "...",
  "length": "detailed",
  "transcriptLanguage": "Spanish",
  "transcriptLanguageCode": "es",
  "summaryLanguage": "English"
}
```

**Error Response:**
```json
{
  "error": "Error message describing what went wrong"
}
```

## Limitations

- ✅ Works with videos that have captions in 19+ supported languages
- ⚠️ Transcript quality depends on Supadata's transcription accuracy and YouTube's caption availability
- ⚠️ Very long videos may be truncated due to OpenAI token limits (max 2000 tokens for comprehensive summaries)
- ⚠️ Some videos may not have publicly available transcripts or captions in the requested language
- ⚠️ Redis connection required for transcript caching (local deployment)
- ⚠️ Translation accuracy depends on OpenAI's multilingual capabilities

## Future Enhancements

- [x] Support for multiple languages ✅ **IMPLEMENTED**
- [ ] Timestamps in summaries linking to video sections
- [ ] Summary history and favorites
- [ ] Batch processing multiple videos
- [ ] Export summaries as PDF
- [ ] Key takeaways and bullet point extraction
- [ ] Language confidence score display
- [ ] Show available caption languages for videos

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `OPENAI_API_KEY` | Your OpenAI API key for GPT-4o-mini model | Yes |
| `SUPADATA_API_KEY` | Your Supadata API key for YouTube transcription | Yes |
| `REDIS_URL` | Redis connection URL (default: redis://localhost:6379) | Yes |

## Troubleshooting

### "Could not fetch transcript" error
- The video may not have captions available in the requested language
- Try auto-detect or a different language
- Try a different video with captions enabled
- YouTube transcripts must be publicly available

### "Invalid YouTube URL" error
- Ensure the URL is a valid YouTube link
- Supported formats: `youtube.com/watch?v=...`, `youtu.be/...`, `youtube.com/embed/...`

### API key errors
- Verify your `OPENAI_API_KEY` and `SUPADATA_API_KEY` are set in `.env.local`
- Check that both API keys are valid and have available credits/quota

### Redis connection errors
- Ensure Redis server is running and accessible at the configured URL
- Check Redis connection string in `REDIS_URL` environment variable
- For local development, Redis should be running on port 6379 by default

## Contributing

Feel free to submit issues and enhancement requests!

## License

MIT
