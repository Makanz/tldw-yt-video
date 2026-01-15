# Quick Start Guide

## Setup (2 minutes)

1. **Get your API key**
   - Visit https://console.anthropic.com/
   - Create an account or log in
   - Go to API keys section
   - Create a new API key and copy it

2. **Configure the project**
   ```bash
   # Create .env.local file in project root
   ANTHROPIC_API_KEY=your_copied_api_key_here
   ```

3. **Run the development server**
   ```bash
   pnpm dev
   ```

4. **Open in browser**
   - Navigate to http://localhost:3000
   - You should see the TLDW YT Video interface

## Using the App

1. **Paste a YouTube URL**
   - Any YouTube video with captions will work
   - Examples:
     - `https://www.youtube.com/watch?v=...`
     - `https://youtu.be/...`

2. **Select summary length**
   - **Brief**: Quick 2-3 paragraph overview
   - **Detailed**: Full analysis in 5-7 paragraphs
   - **Comprehensive**: Complete coverage with all details

3. **Click "Generate Summary"**
   - Wait for the AI to process (usually 5-15 seconds)
   - Your summary will appear below

4. **Copy or share**
   - Use the "Copy to Clipboard" button to copy the summary
   - Share with friends or save for later

## Troubleshooting

### API Key Issues
- Verify the key is correctly copied (no extra spaces)
- Check your API key has available credits
- Restart the dev server after adding the key

### Transcript Not Available
- Some videos don't have public captions
- Try selecting a different video
- Educational content usually has better transcripts

### Slow Response
- First run takes longer (cold start)
- Very long videos may take 20+ seconds
- Check your internet connection

## Project Structure

```
tldr-yt-video/
├── app/
│   ├── api/
│   │   └── summarize/
│   │       └── route.ts          # API endpoint
│   ├── layout.tsx                 # Root layout
│   ├── page.tsx                   # Main UI page
│   └── globals.css                # Tailwind styles
├── .env.example                   # Environment template
├── package.json                   # Dependencies
├── tsconfig.json                  # TypeScript config
└── README.md                      # Full documentation
```

## Development Commands

```bash
# Start development server
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start

# Run TypeScript type check
pnpm tsc --noEmit

# Run linter
pnpm lint
```

## Next Steps

- Customize the UI colors in `app/page.tsx`
- Adjust max token limits in `app/api/summarize/route.ts`
- Add authentication to save user preferences
- Deploy to Vercel for free hosting

## Getting Help

- Check README.md for detailed documentation
- Review error messages carefully
- Verify API key is set correctly
- Try a different video if transcript issues occur
