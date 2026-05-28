# Deployment Guide for FastAPI Blog

This guide covers setting up GitHub Actions and deploying your FastAPI blog to various platforms.

## Prerequisites

1. **GitHub Repository**: Push your code to GitHub
   ```bash
   git remote add origin https://github.com/prasad-nimbalkar/fastapi_blog.git
   git branch -M main
   git push -u origin main
   ```

2. **Create `.env.example`** (commit this, not `.env`):
   ```
   SECRET_KEY=your-secret-key-here
   ALGORITHM=HS256
   ACCESS_TOKEN_EXPIRE_MINUTES=30
   MAX_UPLOAD_SIZE_BYTES=5242880
   ```

## GitHub Actions Setup

### 1. Enable GitHub Actions

1. Go to your repository on GitHub
2. Click **Settings** → **Actions** → **General**
3. Ensure "Allow all actions and reusable workflows" is selected

### 2. Set Up Secrets (for Deployment)

1. Go to **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Add the secrets based on your chosen deployment platform (see below)

## Deployment Options

### Option A: Railway (Recommended - Easiest)

Railway is the easiest option with automatic deployments from Git.

**Setup Steps:**

1. **Create Railway Account**: Go to [railway.app](https://railway.app) and sign up
2. **Create New Project**: Click "New Project" → "Deploy from GitHub repo"
3. **Connect GitHub**: Authorize Railway and select your repository
4. **Configure Environment Variables**:
   - Go to **Variables** tab
   - Add `SECRET_KEY`, `ALGORITHM`, `ACCESS_TOKEN_EXPIRE_MINUTES`, etc.
   - Add `PYTHON_VERSION=3.11`
   - Add `PORT=8000`

5. **Set Start Command**:
   - In your Railway project, go to **Settings**
   - Set **Start Command** to:
     ```
     uv sync && uv run fastapi dev main.py --host 0.0.0.0 --port $PORT
     ```

6. **Get Railway Token** (for GitHub Actions):
   - Go to [railway.app/account/tokens](https://railway.app/account/tokens)
   - Create a new token
   - Copy it

7. **Add to GitHub Secrets**:
   - In GitHub repo: **Settings** → **Secrets and variables** → **Actions**
   - Click **New repository secret**
   - Name: `RAILWAY_TOKEN`
   - Value: Paste your Railway token

8. **Deploy**:
   - Railway auto-deploys from `main` branch
   - Or manually trigger GitHub Actions workflow

### Option B: Render

Render offers good free tier with automatic deployments.

**Setup Steps:**

1. **Create Render Account**: Go to [render.com](https://render.com) and sign up
2. **Create New Web Service**:
   - Click **New** → **Web Service**
   - Connect your GitHub account
   - Select your `fastapi_blog` repository
   - Name your service
   - Environment: `Python 3.11`
   - Build Command: `pip install uv && uv sync`
   - Start Command:
     ```
     uv run fastapi dev main.py --host 0.0.0.0 --port 8000
     ```

3. **Configure Environment Variables**:
   - Go to **Environment**
   - Add your `.env` variables: `SECRET_KEY`, `ALGORITHM`, etc.

4. **Get API Key** (for GitHub Actions):
   - Go to **Account Settings** → **API Keys**
   - Create a new key
   - Get your Service ID from the URL: `https://dashboard.render.com/web/srv-xxxxx`

5. **Add to GitHub Secrets**:
   - `RENDER_SERVICE_ID`: Your service ID
   - `RENDER_API_KEY`: Your API key

### Option C: Heroku (Legacy - Being Phased Out)

⚠️ **Note**: Heroku free tier ended. You'll need a paid account.

### Option D: Docker + Manually

**Build and Run Locally**:

```bash
docker build -t fastapi-blog .
docker run -p 8000:8000 \
  -e SECRET_KEY=your-secret-key \
  -e ALGORITHM=HS256 \
  fastapi-blog
```

**Deploy to Docker Registry**:

```bash
# Build for your platform (e.g., Docker Hub)
docker build -t your-username/fastapi-blog:latest .
docker push your-username/fastapi-blog:latest
```

## GitHub Actions Workflows

### CI Workflow (`.github/workflows/ci.yml`)

Automatically runs on every push and PR to `main` and `develop`:
- Tests Python 3.11 and 3.12
- Runs linting and type checking
- Validates dependencies

### Deploy Workflows

**For Railway** (`.github/workflows/deploy-railway.yml`):
- Automatically deploys to Railway when code is pushed to `main`
- Requires `RAILWAY_TOKEN` secret

**For Render** (`.github/workflows/deploy-render.yml`):
- Automatically deploys to Render when code is pushed to `main`
- Requires `RENDER_SERVICE_ID` and `RENDER_API_KEY` secrets

## Database Setup for Deployment

Your app uses SQLite (`blog.db`). For production:

**Option 1: Use SQLite with Persistent Storage**
- Railway: SQLite persists automatically
- Render: Attach a persistent disk at `/app/data`

**Option 2: Use PostgreSQL (Recommended)**

1. Provision PostgreSQL from your deployment platform
2. Update `database.py` connection string
3. Update your workflow to run migrations if needed

## Testing Your Setup

1. **Make a test commit**:
   ```bash
   git add .
   git commit -m "Add GitHub Actions and deployment config"
   git push origin main
   ```

2. **Check GitHub Actions**:
   - Go to your repo → **Actions** tab
   - Watch the CI workflow run
   - If deployment is configured, watch the deploy workflow

3. **Check Deployment**:
   - Railway: Check dashboard for logs
   - Render: Check dashboard for logs and deployment status

## Troubleshooting

### Deployment Fails

**Check logs**:
- Railway: Dashboard → Logs
- Render: Dashboard → Logs

**Common issues**:
- Missing environment variables: Add them in platform dashboard
- Python version mismatch: Ensure Python 3.11+ is used
- Dependencies not installed: Check `pyproject.toml` is committed

### Actions Don't Trigger

1. Check branch protection rules
2. Verify `.github/workflows/` files are committed
3. Go to **Settings** → **Actions** → ensure workflows are enabled

## Next Steps

1. ✅ Commit all files: `git add . && git commit -m "Setup CI/CD"`
2. ✅ Choose a deployment platform (Railway recommended)
3. ✅ Set up environment variables
4. ✅ Push to main branch
5. ✅ Monitor deployment

## Environment Variables for Production

Make sure these are set in your deployment platform:

```
SECRET_KEY=<generate-a-strong-secret>
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
MAX_UPLOAD_SIZE_BYTES=5242880
```

Generate a strong `SECRET_KEY`:
```bash
python -c "import secrets; print(secrets.token_urlsafe(32))"
```

---

**Need help?** Check the logs in your deployment platform's dashboard!
