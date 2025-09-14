# GitHub Repository Backup Action Template

This template allows you to backup multiple GitHub repositories (private/public) into `.bundle` files and upload them to AWS S3 using GitHub Actions.

**What is backed up**
- All commit history (including all branches and tags)
- Branch and tag information
- Remote configuration and reference information

**What is not backed up**
- History of Issues, Pull Requests, Projects, Discussions, and Actions
- Wiki or static files hosted on GitHub Pages (requires separate export)
- Repository settings (e.g., Branch Protection rules)

**Note**: This workflow is intended to be used with **private repositories**.
Scheduled workflows in public repositories may be automatically disabled if no repository activity occurs for 60 days, so use caution if applied to public repositories.

Reference: [Disabling and enabling a workflow](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/disable-and-enable-workflows)

## Prerequisites

1. **Create an S3 bucket**
   - Create a new S3 bucket in AWS for backups.
   - The bucket name will be used in the workflow YAML.
2. **Create an IAM user**
   - Create a backup IAM user.
   - Create an IAM user's Access Key ID and Secret Access Key.
   - Attach the following policy:
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "s3:PutObject",
                    "s3:GetObject",
                    "s3:ListBucket"
                ],
                "Resource": [
                    "arn:aws:s3:::your-bucket-name",
                    "arn:aws:s3:::your-bucket-name/*"
                ]
            }
        ]
    }
    ```
3. **Create a Personal Access Token (classic)**
   - Login to GitHub
   - Go to `Settings` of your profile → `Developer settings` → `Personal access tokens` → `Tokens (classic)` → `Generate new token`
   - Select `repo` scope (Full control of private repositories)

4. **Set GitHub Secrets**
   - Go to your repository → `Settings` → `Secrets and variables` → `Actions` → `New repository secret`
   - Add the following secrets:
     - `AWS_ACCESS_KEY_ID`: IAM user's Access Key ID
     - `AWS_SECRET_ACCESS_KEY`: IAM user's Secret Access Key
     - `TOKEN_PAT_GITHUB`: Personal Access Token (classic)

## Update YAML
- Replace `your-username` with your GitHub username
- Replace `your-bucket-name` with your S3 bucket name
- List the repositories you want to backup
    ```yaml
    # List of repositories to backup
    for repo in \
      https://github.com/your-username/repo1.git \
      https://github.com/your-username/repo2.git \
      https://github.com/your-username/repo3.git
    do
    ```
## Schedule change

- Modify the `cron` value in YAML to change the workflow schedule
  - Example: Every Wednesday at 0:00 UTC:
    ```yaml
    schedule:
    - cron: "0 0 * * 3"
    ```
- Schedule is in UTC; adjust for your local timezone as needed

## Usage

- The workflow runs automatically on the schedule
- Manual run: click `Run workflow` from the [Actions] tab

## Restore

1. Download the backup `.bundle` file
2. Clone and restore:
    ```bash
    git clone repo-name-YYYYMMDD.bundle repo-name
    cd repo-name
    git remote add origin https://github.com/your-username/repo-name.git
    git push -u origin --all
    git push origin --tags
    ```
