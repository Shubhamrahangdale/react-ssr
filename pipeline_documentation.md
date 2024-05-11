
# CI/CD Pipeline Documentation

This document outlines the steps to set up and trigger the CI/CD pipeline for a React application using GitHub Actions and AWS.

## Setup Process

1. **Create Workflow File**: Create a workflow file (e.g., `.github/workflows/main.yml`) in your repository with the following content:

```
name: CI/CD Pipeline
on:
  push:
    branches:
      - master

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
        
      - name: Uninstall Node.js and npm
        run: |
          sudo apt-get remove --purge nodejs npm
          sudo apt-get autoremove
          sudo apt-get autoclean

      - name: Setup Node.js and npm
        uses: actions/setup-node@v3
        with:
          node-version: '14.21.3'
          npm: '6.14.8'

      - name: Install Build Essential
        run: |
          sudo apt-get update
          sudo apt-get install -y build-essential

      - name: Install Python 2.7
        run: |
          sudo apt-get install python2.7
          sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1

      - name: Clean npm cache
        run: |
          npm cache clean

      - name: Install dependencies
        run: |
          npm install npm@6.14.8
          npm install

      - name: Build
        run: npm run build

      - name: Deploy to AWS
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      - run: |
          aws s3 sync build/ s3://<s3-bucket-name>/
          aws cloudfront create-invalidation --distribution-id <cloudfront-distribution-id> --paths "/*"
````

3. Add AWS Credentials: Add your AWS access key ID and secret access key as secrets in your GitHub repository under Settings > Secrets.

 ## Triggering the Pipeline
 The pipeline will automatically trigger on every push to the master branch.

 Alternatively, you can trigger the pipeline manually by going to the Actions tab in your GitHub repository, selecting the workflow, and clicking the "Run workflow" button.

 ## Additional Information

 The pipeline uses npm install to install dependencies and npm run build to build the React application.
 The aws-actions/configure-aws-credentials GitHub Action is used to configure AWS credentials for deployment.
 The aws s3 sync command is used to sync the build output to the S3 bucket, and the aws cloudfront create-invalidation command is used to create a CloudFront invalidation to update the CDN cache.
