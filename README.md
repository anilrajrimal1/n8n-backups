# n8n Workflow Backup 

This repository is designed to help you automatically back up your n8n workflows at regular intervals. By using this setup, you can ensure that your workflows are safe and secure, even in the event of unexpected data loss or system failures.

![Image](https://github.com/anilrajrimal1/n8n-backups/blob/master/Screenshot%20from%202024-05-09%2015-24-02.png)

## Setup Instructions

Follow these steps to set up automated backups for your n8n workflows:

1. **Create a Backup Repository**: Start by creating a backup repository to your GitHub account.

2. **Generate GitHub Token**: Go to [GitHub settings](https://github.com/settings/tokens/new) and generate a new token with the `repo` scope selected. Copy this token for later use.

3. **Configure new n8n workflow**: In your n8n instance, set up a workflow with the template i provided here.

5. **Create Required Cerdentials** Create credentials like API forfor GitHub with the previous generated token. Also create a n8n API from the ` Settings > n8n api > generate `

4. **Just Modify (Compulsory)**: If you want to change the backup frequency, you can modify the schedule, you also need to change the user, repo name etc as well.


## Restoring Workflows

### For restoration I personally suggest you to download the JSON file from GitHub on your local machine and then import it from n8n.

## Contributing

Contributions are welcome! If you have any suggestions, improvements, or feature requests, feel free to open an issue or submit a pull request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
