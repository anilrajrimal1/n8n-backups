# n8n Workflow Backups

A ready-to-import n8n workflow that backs up every workflow on your n8n instance to a GitHub repository. Each workflow is stored as its own JSON file, and a commit is made only when a workflow is new or has actually changed, so the repository history doubles as a change log for your automations.

![n8n backup workflow](Screenshot%20from%202024-05-09%2015-24-02.png)

## Features

- Backs up **all** workflows through the n8n API, one file per workflow
- Commits only on change: unchanged workflows are skipped, so there is no commit noise
- Readable diffs: JSON keys are sorted before saving, so commits show real edits instead of reordered fields
- One place to configure: owner, repository and folder all live in the `Globals` node
- Runs hourly on a schedule, or on demand with the manual trigger
- No custom code to host: plain n8n nodes plus one small Code node

## How it works

```
Every hour / Run manually
        │
     Globals ──────────────► repo.owner, repo.name, repo.path
        │
       n8n ────────────────► list every workflow on the instance
        │
   OneAtATime ─────────────► loop, one workflow per pass
        │
     GitHub (get file) ────► fetch <repo.path>/<workflow name>.json, if it exists
        │
     Merge + isDiffOrNew ──► sort keys, compare stored vs live
        │
  check_github_status
     ├── same       → skip
     ├── different  → GitHub Edit   (commit: "update: n8n backup <name>.json")
     └── new        → GitHub Create (commit: "Add: n8n backup file named <name>.json")
```

## Requirements

- An n8n instance (self-hosted or cloud) with the **public API** enabled
- A GitHub repository to store backups in. A **private** repository is strongly recommended, see [Security](#security)
- A GitHub personal access token for that repository

## Setup

### 1. Create the backup repository

Create an empty repository on GitHub, for example `n8n-backups`. Initialize it with a README so the default branch exists.

### 2. Create a GitHub token

Use either kind of token:

| Token type | Permissions needed |
|---|---|
| Fine-grained (recommended) | Access to the backup repository only, **Contents: Read and write** |
| Classic | `repo` scope |

### 3. Create an n8n API key

In n8n, open **Settings > n8n API**, click **Create an API key** and copy it.

### 4. Import the workflow

1. Download [`workflows/n8n-backup.json`](workflows/n8n-backup.json).
2. In n8n, create a new workflow, open the **⋯** menu and choose **Import from File**.

### 5. Add credentials

| Node | Credential type | Value |
|---|---|---|
| `n8n` | n8n API | API key from step 3, and your instance's base URL ending in `/api/v1` |
| `GitHub`, `GitHub Edit`, `GitHub Create` | GitHub API | Token from step 2 |

### 6. Point it at your repository

Open the **Globals** node and set:

| Field | Example | Notes |
|---|---|---|
| `repo.owner` | `your-github-username` | User or organization that owns the repository |
| `repo.name` | `n8n-backups` | Repository name only, not the URL |
| `repo.path` | `workflows/` | Folder for the files. Keep the trailing `/` |

All three GitHub nodes read these values, so this is the only place you need to edit.

### 7. Test, then activate

1. Click **Test workflow**. The first run creates one file per workflow.
2. Check your repository, then run it again: nothing should be committed, because nothing changed.
3. Toggle the workflow to **Active**.

## Customization

- **Schedule:** open the `Every hour` node and change the interval. Hourly suits most setups; every few minutes adds API calls without much benefit.
- **Folder layout:** change `repo.path`, for example `backups/prod/` to keep several instances in one repository.
- **Commit messages:** edit the `Commit Message` field in `GitHub Edit` and `GitHub Create`.
- **Notifications:** connect a Slack, Email or Telegram node after `GitHub Edit` or `GitHub Create` to be told when a workflow changes.

## Restoring a workflow

1. Open the workflow's JSON file in your backup repository and download it (**Download raw file**).
2. In n8n, create a new workflow and choose **Import from File**.
3. Re-select credentials on any nodes that use them, then save.

To roll back to an older version, open the file's **History** on GitHub, pick the commit you want, and download that version instead.

## Security

- **Credential secrets are not exported.** n8n stores secrets separately, and the API returns only credential names and IDs.
- **Everything else in a workflow is.** That includes URLs, hostnames, S3 bucket names, shell commands, hardcoded values in Code nodes and any pinned test data. Keep the backup repository **private** unless you are sure none of your workflows contain anything sensitive.
- Give the GitHub token access to the backup repository only, and set an expiry date.

## Troubleshooting

| Problem | Likely cause |
|---|---|
| `401` on the `n8n` node | Wrong API key, or the base URL is missing `/api/v1` |
| `404` on `GitHub Edit` or `GitHub Create` | `repo.owner` or `repo.name` is wrong, or the token cannot see the repository |
| `403` or `Resource not accessible` | Token lacks write access to repository contents |
| Every run commits every workflow | The stored files were edited by hand. Let one run overwrite them, then it settles |
| A renamed workflow leaves an old file behind | Files are named after the workflow. Delete the old file manually |
| Files appear in unexpected subfolders | A workflow name contains `/`. Rename the workflow |

## Contributing

Issues and pull requests are welcome.

## License

[MIT](LICENSE)
