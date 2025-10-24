# NAIC NPN Lookup — Push to GitHub

These steps push this app to your GitHub repository using **HTTPS**.

> Repo you mentioned: `https://github.com/qualifierCaj198/NAIC`

## 1) Prepare local repo
From the project folder (this folder), run:
```bash
git init
git add .
git commit -m "Initial commit: NAIC NPN lookup app"
git branch -M main
```

## 2) Add GitHub remote
```bash
git remote add origin https://github.com/qualifierCaj198/NAIC.git
```

If the remote already exists and points elsewhere:
```bash
git remote remove origin
git remote add origin https://github.com/qualifierCaj198/NAIC.git
```

## 3) Push to GitHub
```bash
git push -u origin main
```

You'll be prompted to sign in with your GitHub account (use a **personal access token** if asked for a password).

## 4) Verify
Open the repo URL in your browser and confirm files appear on the `main` branch.

---

### Optional: .gitignore (already included or create it)
We recommend ignoring Python venvs and caches. If you need a custom one, create a `.gitignore` like:
```
.venv/
__pycache__/
*.pyc
*.pyo
*.pyd
.Python
env/
build/
dist/
*.log
.DS_Store
```

### Optional: Run locally
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python app.py
# open http://127.0.0.1:5000/
```

---

_Last updated: 2025-10-24T14:17:00.603151Z_

---

## GitHub Actions auto-deploy (password SSH)

1) In your GitHub repo, go to **Settings → Secrets and variables → Actions → New repository secret** and add:
   - `SSH_HOST` = your server IP (e.g., `45.77.167.197`)
   - `SSH_USER` = `root` (or another user with permission to manage systemd & nginx)
   - `SSH_PASSWORD` = your SSH password
   - `APP_DIR` = `/opt/naic`

2) Commit the workflow:
   - The file is already included at `.github/workflows/deploy.yml`.
   - On each push to `main`, GitHub will:
     - Create a tarball of the repo
     - Upload it to `${APP_DIR}/releases`
     - Unpack to `${APP_DIR}/app`
     - Create/activate a Python venv, install requirements, run Gunicorn
     - Create a systemd service `naic` if missing, and restart it
     - Create an Nginx site on port 80 pointing to Gunicorn

3) Deploy by pushing to `main`:
```bash
git add -A
git commit -m "Trigger deploy"
git push
```

4) Visit `http://<your-ip>/` to see the app.

> You can later harden security by switching to SSH keys. For now this workflow uses password-based SSH as requested.

---

## Enable HTTPS (Let's Encrypt via the workflow)

Set these **repo secrets** in GitHub → Settings → Secrets and variables → Actions:

- `SSH_HOST` = your server IP (e.g., 45.77.167.197)
- `SSH_USER` = `root` (or privileged user)
- `SSH_PASSWORD` = your SSH password
- `APP_DIR` = `/opt/naic`
- `SERVER_NAME` = your domain (e.g., `naic.example.com`)
- `LETSENCRYPT_EMAIL` = your email (for cert renewal notices)

**DNS:** Point an A record for `SERVER_NAME` → your server IP.  
On your next push to `main`, the workflow will:
1. Deploy the app
2. Configure Nginx for HTTP
3. Run Certbot to obtain a TLS certificate and enable **HTTPS + HTTP→HTTPS redirect**

If `SERVER_NAME` is unset, it will run HTTP-only on port 80.