# n8n Workflow Backups

An n8n workflow that backs up every workflow on your n8n instance to a GitHub repository on a schedule. Each workflow is saved as its own JSON file, and a commit is made only when a workflow is new or has changed.

![n8n backup workflow](Screenshot%20from%202024-05-09%2015-24-02.png)

## How it works

1. A schedule trigger runs the workflow every 10 minutes.
2. The **n8n** node fetches all workflows from your instance through the n8n API.
3. For each workflow, the **GitHub** node looks for an existing file at `workflows/<workflow name>.json`.
4. The **isDiffOrNew** code node compares the two and routes each workflow:
   - **new**: creates the file
   - **different**: updates the file
   - **same**: skips it

The template is [`workflows/n8n-backup.json`](workflows/n8n-backup.json).

## Setup

1. **Create a backup repository** on GitHub. It can be private.
2. **Import the template.** In n8n, create a new workflow and import `workflows/n8n-backup.json`.
3. **Create an n8n API credential.** In n8n, go to **Settings > n8n API** and create an API key. Add it as an n8n API credential and select it on the **n8n** node.
4. **Create a GitHub credential.** The GitHub nodes use **GitHub OAuth2**. Create that credential, or switch the nodes to a GitHub access token with write access to your backup repository.
5. **Point it at your repository.** In the **Globals** node, set:
   - `repo.owner`: your GitHub username or organization
   - `repo.name`: your backup repository
   - `repo.path`: the folder the files are written to (default `workflows/`)

   All three GitHub nodes read these values, so this is the only place to change.
6. **Adjust the schedule** if 10 minutes is too often, then activate the workflow.

## Restoring a workflow

Download the workflow's JSON file from your backup repository, then import it in n8n with **Import from File**.

## Contributing

Suggestions and improvements are welcome. Open an issue or a pull request.

## License

[MIT](LICENSE)
