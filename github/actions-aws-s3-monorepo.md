## Github Actions to Deploy to AWS S3 Bucket

Yes, you can absolutely use GitHub Actions to deploy files to an Amazon S3 bucket. This is a common use case, especially for deploying static websites or build artifacts. Here’s a high-level overview of how you can do it:

✅ Prerequisites
An AWS S3 bucket created.

AWS access credentials (AWS Access Key ID and Secret Access Key) with permissions to upload to the bucket.

Secrets set up in GitHub:

AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

You can add these secrets in your repository under:
Settings → Secrets and variables → Actions → New repository secret

🧱 Example GitHub Actions Workflow
Here's a basic example of a GitHub Actions workflow that syncs files to S3:

```
# .github/workflows/deploy-to-s3.yml
name: Deploy to S3

on:
  push:
    branches:
      - main  # Or your deployment branch

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1  # change as needed

    - name: Sync files to S3
      run: |
        aws s3 sync ./dist s3://your-bucket-name --delete

```

- ./dist should be the path to your build/output directory.
- The --delete flag ensures that files deleted locally are also removed from the S3 bucket.

🔒 IAM Permissions Needed
Make sure the IAM user or role used in the GitHub secrets has at least the following permissions:

```
./dist should be the path to your build/output directory.

The --delete flag ensures that files deleted locally are also removed from the S3 bucket.

🔒 IAM Permissions Needed
Make sure the IAM user or role used in the GitHub secrets has at least the following permissions:
```

## Github Actions in a Mono Repo

How GitHub Actions Works in a Monorepo
GitHub Actions runs workflows based on events (like push, pull_request, etc.), but you can target specific folders or files using the paths or paths-ignore filters. This is ideal for monorepos.

🔧 Example: Run Only When a Subdirectory Changes
Suppose you have this monorepo structure:

```
/my-monorepo
  /frontend
  /backend
  /shared
```

You can trigger workflows only when something in frontend/ changes:
```
# .github/workflows/frontend-deploy.yml
name: Frontend Deployment

on:
  push:
    branches:
      - main
    paths:
      - 'frontend/**'

jobs:
  deploy-frontend:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Build and Deploy Frontend
        run: |
          cd frontend
          npm install
          npm run build
          # deploy steps...
```

Similarly, a separate workflow can handle backend/** changes.

🧠 Tips for Monorepos + GitHub Actions
- Use paths to isolate workflow triggers per project.
- Use matrix builds if you want to run the same logic across multiple directories.
- Modularize workflows using reusable workflows.
- Cache dependencies per project using key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }} or similar logic scoped to subfolders.