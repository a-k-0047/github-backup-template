# GitHub Repository Backup Action Template

This repository provides a GitHub Actions workflow to back up multiple repositories (private/public) to AWS S3 as `.bundle` files.

## Documentation

- [English Documentation](./docs/en/README.md)
- [日本語ドキュメント](./docs/ja/README.md)

## Features
- Backup multiple repositories at once
- Store backups in AWS S3
- Private repository support using Personal Access Token (classic)

## Note
It is recommended to use this workflow in **private repositories**.  
Scheduled workflows in public repositories may be automatically disabled if there is no activity for 60 days.
