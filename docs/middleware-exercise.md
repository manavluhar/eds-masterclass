# Middleware Exercise: Cloudflare Worker with Cat Viewer Block

This exercise demonstrates how to use a Cloudflare Worker as middleware to securely proxy API requests from your Edge Delivery Services site.

## What is Middleware?

Middleware acts as an intermediary between your browser and backend APIs:

```
┌──────────┐         ┌──────────────┐         ┌──────────────┐
│ BROWSER  │────────▶│ MIDDLEWARE   │────────▶│ API ENDPOINT │
│          │ Request │              │ Request │              │
│          │◀────────│              │◀────────│              │
└──────────┘ Response└──────────────┘ Response└──────────────┘
                      (Token Exchange)
                      (Secrets Handling)
```

**Why use it?**
- Keep API keys and secrets server-side (never exposed to browsers)
- Handle CORS for cross-origin requests
- Stitch content from multiple APIs
- Connect to private/customer network endpoints
- Transform requests/responses, add caching, rate limiting

## Getting Started

**Branch:** `middleware`

**Prerequisites:**
1. Cloudflare account: https://dash.cloudflare.com/
2. Wrangler CLI: `npm install -g wrangler` (or use `npx`)
3. Cat API key: https://thecatapi.com/signup

**Project Structure:**
- Worker: `middleware-worker/` directory
- Block: `blocks/cat-viewer/` directory

## Exercise Steps

### Step 1: Set Up the Worker

1. Navigate to worker directory and install dependencies:
   ```bash
   cd middleware-worker
   npm install
   ```

2. Set your Cat API key as a secret:
   ```bash
   wrangler secret put CAT_API_KEY
   ```

3. Test locally:
   ```bash
   npm run dev
   ```
   Worker runs at `http://localhost:8787`

### Step 2: Deploy the Worker (Optional)

```bash
npm run deploy
```

Note your worker URL: `https://middleware-worker.YOUR_SUBDOMAIN.workers.dev`

### Step 3: Test Locally

1. Start your EDS dev server:
   ```bash
   aem up
   ```

2. Create a page in Document Authoring:
   - Add a `cat-viewer` block
   - Add a link with the worker URL (`http://localhost:8787`)
   - Preview the page

3. Test the block:
   - Verify cat images load
   - Test breed selector, refresh button, and info modal

**For production:** Update the link in the block to use your deployed worker URL.

## Next Steps

Here are concrete features you can add that require changes to both the block and worker:

**Search by Breed Name:**
- Block: Add a search input field to filter breeds by name
- Worker: Add `/cats/search?q=siamese` endpoint that searches breed names and returns matching cats

**Favorites/Bookmarking:**
- Block: Add a "Favorite" button to save favorite cats
- Worker: Add `/favorites` endpoints (GET/POST/DELETE) using Cloudflare KV to store user favorites

**Breed Comparison:**
- Block: Add UI to select and compare multiple breeds side-by-side
- Worker: Add `/cats/compare?breeds=beng,siam` endpoint that fetches and combines data from multiple breeds

**Image Filters:**
- Block: Add controls for image size, format, or filters
- Worker: Add query params to transform image URLs or proxy through image transformation service
