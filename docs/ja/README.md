# GitHub リポジトリ バックアップアクション テンプレート

このテンプレートは、複数の GitHub リポジトリ（プライベート/パブリック）を `.bundle` ファイルとしてバックアップし、AWS S3 に保存する GitHub Actions ワークフローテンプレートです。

**バックアップされるもの**
- すべてのコミット履歴（全ブランチ・全タグを含む）
- ブランチ・タグ情報
- リモート設定・リファレンス情報

**バックアップされないもの**
- Issues / Pull Requests / Projects / Discussions / Actions の履歴
- Wiki や GitHub Pages に置いた静的ファイル（別途エクスポートが必要）
- リポジトリの設定（Branch Protection ルールなど）

**注意**: このワークフローは**プライベートリポジトリ**での運用を想定しています。
公開リポジトリでスケジュール実行すると、過去 60 日間アクティビティがない場合に自動で無効化されることがありますのでご注意ください。

参考：[ワークフローの無効化と有効化](https://docs.github.com/ja/actions/how-tos/manage-workflow-runs/disable-and-enable-workflows)

## 事前準備
1. **S3 バケットの作成**
   - AWS にてバックアップ用の S3 バケットを作成します。
   - バケット名は後で YAML 内で使用します。

2. **IAM ユーザー作成**
   - バックアップ用 IAM ユーザーを作成します。
   - IAM ユーザーのアクセスキーID、シークレットキーを作成します。
   - IAM ユーザーに以下の権限を付与します:
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

3. **Personal Access Token (classic) の作成**

   - GitHub アカウントでログイン
   - プロフィールの`Settings` → `Developer settings` → `Personal access tokens` → `Tokens (classic)` → `Generate new token`
   - Scope に `repo` (Full control of private repositories) を選択

4. **GitHub Secrets の設定**

   - GitHub リポジトリ画面  → `Settings` → `Secrets and variables` → `Actions` → `New repository secret`
   - 以下の Secrets を追加:
     - `AWS_ACCESS_KEY_ID` : IAM ユーザーのアクセスキーID
     - `AWS_SECRET_ACCESS_KEY` : IAM ユーザーのシークレットキー
     - `TOKEN_PAT_GITHUB` : GitHub の Personal Access Token (classic)


## YAML の設定変更
- `your-username` を自分の GitHub アカウント名に変更
- `your-bucket-name` を作成した S3 バケット名に変更
- バックアップ対象のリポジトリ URL を列挙
    ```yaml
    # List of repositories to backup
    for repo in \
      https://github.com/your-username/repo1.git \
      https://github.com/your-username/repo2.git \
      https://github.com/your-username/repo3.git
    do
    ```

## 起動スケジュール変更方法

- YAML 内の `cron` の値を変更すると実行スケジュールを変更可能
  - 例: 毎週水曜 0:00 UTC にしたい場合:
    ```yaml
    schedule:
    - cron: "0 0 * * 3"
    ```
- UTC で指定するため、日本時間 JST に合わせる場合は +9時間を考慮

## 使用方法

- ワークフローは自動でスケジュール通り実行されます
- 手動で実行する場合は [Actions] タブから `Run workflow` をクリック

## 復元方法

1. バックアップした `.bundle` をローカルに取得
2. クローンして復元:
    ```bash
    git clone repo-name-YYYYMMDD.bundle repo-name
    cd repo-name
    git remote add origin https://github.com/your-username/repo-name.git
    git push -u origin --all
    git push origin --tags
    ```
