# Technopie — Self-Hosted Deployment Guide

## Requirements

- Node.js 18 or higher
- PostgreSQL 14 or higher
- A valid OpenAI API key (for the AI chat assistant)

---

## 1. Upload Files

Upload the entire `technopie-production/` folder to your server.

Your server directory should look like this:

```
technopie/
├── index.cjs        ← Node.js server (handles API + serves frontend)
├── public/          ← React frontend static files
│   ├── index.html
│   └── assets/
├── package.json
├── .env             ← create this from .env.example
└── .env.example
```

---

## 2. Set Up Environment Variables

Copy `.env.example` to `.env` and fill in your values:

```bash
cp .env.example .env
nano .env
```

Required variables:

| Variable | Description |
|---|---|
| `PORT` | Port the server will listen on (e.g. `3000`) |
| `DATABASE_URL` | PostgreSQL connection string |
| `OPENAI_API_KEY` | Your OpenAI API key |
| `NODE_ENV` | Must be set to `production` |

---

## 3. Set Up the Database

Run these SQL statements in your PostgreSQL database to create the required tables:

```sql
CREATE TABLE IF NOT EXISTS contact_submissions (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  email TEXT NOT NULL,
  company TEXT NOT NULL,
  project_requirement TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW() NOT NULL
);
```

---

## 4. Start the Server

```bash
node index.cjs
```

Or use npm start:

```bash
npm start
```

The server will start on the port defined in `.env` (default: `3000`).
Open `http://your-server-ip:3000` in your browser.

---

## 5. Run with PM2 (Recommended for Production)

PM2 keeps your server running and restarts it automatically on crashes:

```bash
# Install PM2 globally
npm install -g pm2

# Start the server
pm2 start index.cjs --name "technopie" --env production

# Auto-start on server reboot
pm2 startup
pm2 save
```

---

## 6. Reverse Proxy with Nginx (Recommended)

If you want to run on port 80/443, add this Nginx config:

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

## Features

- **Homepage** — Hero, Services, Portfolio, Why Choose Us, Case Study, Testimonials, Blog, Contact
- **Services Page** — Detailed service descriptions
- **Portfolio Page** — Project showcase with filters
- **Case Studies Page** — Client success stories
- **Blog Page** — Data analytics & AI articles
- **About Page** — Team, mission, and company story
- **Contact Page** — Form submissions saved to PostgreSQL
- **AI Chat Assistant** — Powered by OpenAI GPT-4o-mini (requires valid API key with credits)

---

## Troubleshooting

**Server won't start:**
- Check `NODE_ENV=production` is set in your `.env`
- Ensure `DATABASE_URL` points to a running PostgreSQL instance
- Verify Node.js version: `node --version` (needs ≥ 18)

**Chat widget returns errors:**
- Ensure `OPENAI_API_KEY` is set correctly
- Check your OpenAI account has billing credits at platform.openai.com/account/billing

**Contact form doesn't save:**
- Run the SQL above to create the `contact_submissions` table
- Check `DATABASE_URL` is correct and the database is accessible
