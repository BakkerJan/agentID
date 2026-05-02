# Agent Identity Discovery Toolkit — Setup Guide

A practical guide to getting the toolkit running locally using Docker and an Entra app registration.

---

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
- [Git](https://git-scm.com/) installed
- Access to an Entra tenant with at least **Application Administrator** or **Global Reader** rights
- A terminal (Mac: Terminal or iTerm; Windows: PowerShell or Windows Terminal)

---

## Step 1: Clone the Repository

Open a terminal and navigate to where you want to store the project, then run:

```bash
git clone https://github.com/microsoft/entra-agentid-samples.git
cd entra-agentid-samples
git checkout feature/migration-toolkit
cd migration-toolkit/toolkit
```

> **Mac tip:** If you're not sure where to put it, `~/Documents/dev/` is a good default. Create it with `mkdir -p ~/Documents/dev` first.

---

## Step 2: Create an App Registration in Entra

1. Go to [Entra admin center](https://entra.microsoft.com) → **Applications** → **App registrations** → **New registration**
2. Give it a name, e.g. `Agent Identity Discovery Toolkit`
3. Leave **Supported account types** as single tenant
4. Click **Register**

### Note down these two values from the Overview page:
- **Application (client) ID**
- **Directory (tenant) ID**

---

## Step 3: Configure Authentication

In your app registration, go to **Authentication**:

1. Click **Add a platform** → select **Mobile and desktop applications**
2. Add this redirect URI:
   ```
   http://localhost:8080/api/auth/callback
   ```
3. Click **Configure** and **Save**
4. Go to the **Settings** tab and enable **Allow public client flows**
5. Click **Save**

> No client secret is needed — the app uses PKCE (public client auth).

---

## Step 4: Configure the Environment File

Copy the example config:

```bash
cp .env.example .env
```

Open `.env` in a text editor:

```bash
open -e .env        # Mac
notepad .env        # Windows
```

Fill in your values:

```env
AZURE_CLIENT_ID=<your-application-client-id>
AZURE_TENANT_ID=<your-tenant-id>
REDIRECT_URI=http://localhost:8080/api/auth/callback
SESSION_SECRET=<generate-a-random-string>
```

To generate a secure `SESSION_SECRET` on Mac/Linux:

```bash
openssl rand -hex 32
```

---

## Step 5: Start the Toolkit

```bash
docker compose up -d
```

Docker will build the image and start the container in the background. Wait a few seconds, then open your browser and go to:

```
http://localhost:8080
```

---

## Step 6: Sign In

1. The app will show your **Tenant ID** and **Application (Client) ID** pre-filled from your `.env`
2. Click **Sign in with Entra**
3. Complete the Microsoft sign-in flow
4. You'll be redirected back to the dashboard

---

## Step 7: Run a Scan

Once signed in, click **Start Scan**. The toolkit will:

1. **Discover** — query Microsoft Graph for all service principals
2. **Enrich** — pull tags, permissions, and metadata
3. **Classify** — identify agentic service principals (Copilot Studio, Azure AI, etc.)

Results appear in the dashboard when the scan completes. You can also export to CSV.

---

## Stopping the Toolkit

```bash
docker compose down
```

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `AADSTS50011` redirect URI mismatch | URI not registered in Entra | Add `http://localhost:8080/api/auth/callback` under Authentication → Mobile and desktop |
| Login page keeps reappearing after sign-in | Session not persisting | Make sure `SESSION_SECRET` is set in `.env` and the container was restarted after editing |
| Wrong tenant or client ID shown | `.env` not loaded | Run `docker compose down` then `docker compose up -d` to reload env |
| Container won't start | Docker not running | Open Docker Desktop and wait for it to fully start |
| `cp .env.example .env` fails | Wrong folder | Run `ls -a` and confirm `.env.example` is present; navigate with `cd` if not |
