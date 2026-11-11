# Daily backup of the n8n workflows to GitHub

`MATTR_daily_backup_to_GitHub.json` runs every night at midnight (India time). It saves each **active** workflow to GitHub as `backups/<workflow name>.json`. It only commits a file when that workflow changed since the last run, so GitHub's history shows exactly what changed, and when.

What it leaves out:

- **Pinned test data and saved state.** They can hold real figures.
- **Passwords and keys.** A workflow only holds the *names* of its credentials, never their secrets.

## Files in this folder

| File | What it is |
|---|---|
| `MATTR_daily_backup_to_GitHub.json` | The n8n workflow. Import this one. |
| `README.md` | This guide. |
| `backup.sql`, `build_files.js`, `plan_changes.js` | **Not used yet.** An earlier, unfinished version that also backs up the bot database's structure (tables and functions, no rows). Not needed for the daily backup. |

## Set it up (about 15 minutes)

### 1. A private repository on GitHub

1. On github.com, click **New repository**. Name it, for example, `mattr-n8n-backup`.
2. Choose **Private** and tick **Add a README file**. The README matters: the workflow can't write to a completely empty repository.
3. Click **Create repository**.

### 2. A GitHub token that can only touch that repository

1. Go to GitHub → your photo → **Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token**.
2. Fill in the token:
   - **Name:** `n8n backup`.
   - **Expiration:** 1 year. Put a reminder in your calendar to renew it.
   - **Repository access:** **Only select repositories**, then pick `mattr-n8n-backup`.
   - **Permissions → Repository permissions → Contents:** **Read and write**.
3. Click **Generate token** and copy it. GitHub shows it only once.

### 3. Two credentials in n8n

- **GitHub:** go to **Overview → Credentials → Create credential → GitHub API**.
  - Name it **GitHub account**.
  - **User:** your GitHub username.
  - **Access Token:** paste the token.
- **n8n API:**
  1. In n8n, go to **Settings → n8n API → Create an API key**. Copy the key.
  2. Go to **Create credential → n8n API**.
     - Name it **n8n API**.
     - **API Key:** paste the key.
     - **Base URL:** `http://localhost:5678/api/v1`.

### 4. Import the workflow and fill it in

1. Go to **Overview → Create workflow → ⋯ → Import from File**, and pick `MATTR_daily_backup_to_GitHub.json`.
2. Open **Backup settings** and replace:
   - `YOUR_GITHUB_USERNAME` with your GitHub username, or the organisation that owns the repository.
   - `YOUR_BACKUP_REPO` with `mattr-n8n-backup`.
   - Leave **branch** as `main` and **folder** as `backups/`.
3. Open **Get many workflows**, **Get file from GitHub**, **Create file** and **Update file**. Pick the credential where the field is red.
4. Click **Save**, then **Execute workflow**. In GitHub you should see a `backups/` folder with one file per active workflow.
5. Run it a second time. It should make **no** new commits, because nothing changed.
6. **Publish** it, so it runs every night.

## Good to know

- **To back up inactive workflows too**, open **Get many workflows** and switch off **Return only active workflows** (under Filters).
- **Renaming a workflow** makes a new file. The old file stays, so delete it in GitHub if you don't want it.
- **If one upload fails**, for example a short GitHub outage, the others still go through. That one is retried the next night. To see failures, look in **Executions** for runs where **Create file** or **Update file** shows an error.
- **To restore a workflow:**
  1. Open its file in GitHub and click **Download raw file**.
  2. In n8n, go to **Create workflow → ⋯ → Import from File**.
  3. Pick the credentials again where a node is red.
  - For an older version, open the file's **History** in GitHub first.

## What this does not cover if the VM is lost

The workflows come back from GitHub, but three things don't:

- **Credentials** (WhatsApp, OpenRouter, Postgres): you create them again by hand, with the same names.
- **The bot's database** (`tally_data`): its tables and functions, who can see which company, invites, and the question log.
- **The company books in it:** these can be sent again from the backend (`python -m app.manage n8n-resync <id>`).
