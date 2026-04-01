# Hosting Guide

This project can be hosted quickly in two ways:

1. **Streamlit Community Cloud** (fastest)
2. **Render/Railway with Docker** (more control)

## Option 1: Streamlit Community Cloud (Recommended)

### Steps

1. Push this repo to GitHub.
2. Go to [https://share.streamlit.io/](https://share.streamlit.io/) and sign in.
3. Click **New app**.
4. Select your repo and branch.
5. Set **Main file path** to `app.py`.
6. Open **Advanced settings -> Secrets** and add:

```toml
OPENAI_API_KEY="your_openai_api_key"
```

7. Deploy.

### Notes

- No need to upload `.env` file to GitHub.
- The app installs dependencies from `requirements.txt`.

## Option 2: Render or Railway (Docker)

This repo now includes a `Dockerfile`, so most container platforms work directly.

### Render

1. Push repo to GitHub.
2. In Render, choose **New + -> Web Service**.
3. Connect repo.
4. Environment: **Docker**.
5. Add environment variable:
   - `OPENAI_API_KEY=your_openai_api_key`
6. Deploy.

### Railway

1. Create new project from GitHub repo.
2. Railway detects Docker and builds automatically.
3. Add environment variable:
   - `OPENAI_API_KEY=your_openai_api_key`
4. Deploy.

## Verify Deployment

After deploying:

1. Open app URL.
2. Upload a small PDF.
3. Click **Process**.
4. Ask: "Give me a short summary of this document."

If this works, hosting is successful.
