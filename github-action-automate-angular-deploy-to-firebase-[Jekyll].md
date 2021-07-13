---
title: "Using Github actions to automate CI/CD and deploy angular 11 apps to firebase"
date: 2021-01-28T13:45:06+06:00
image: https://github.com/superwalnut/mrkevin.wang/blob/master/images/md-gist.png
feature_image: https://github.com/superwalnut/mrkevin.wang/blob/master/images/md-gist-feature.png
author: Kevin Wang
summary: I am building an angular 11 app and deploy to firebase using github actions to automate this process.
---

If you have built an angular app, where would you deploy it? I have tried a number of cloud providers which you can achieve the same results, such as aws lambda, azure web app, netlify and google firebase, etc. I am writing a series of articles to demonstrate how to deploy in all of these providers. Today I am showing you how to build an angular app and deploy to google firebase.

## Part 1. Prerequisite

### Setup [node.js](https://nodejs.org/en/download/) environment and install angular cli.

{% gist 0b5172eec09c81931da49dd338395baf %} bash
ng new test-app
{% gist 1f7ddbe10f18fde7af49bb0644b04b4c %} bash
npm install -g firebase-tools
{% gist 76fe2910bf7c8d7d2bbae06231ba07a6 %} bash
firebase login
{% gist d77d5bd675cab891e2160b9cabaf8e06 %} bash
firebase init
{% gist 52283ae636b8dfd328aed2a94b781fec %}
Database: Deploy Firebase Real-time Database Rules
Firestore: Deploy rules and create indexes for Firestore
Functions: Configure and deploy Cloud Functions
Hosting: Configure and deploy Firebase Hosting sites
Storage: Deploy Cloud Storage security rules
{% gist a6be7e526107a4fc57ade2ec804a42a2 %}
 Set up automatic builds and deploys with GitHub?
{% gist bb19b394e8698e47f2ce286b93622054 %} bash
https://github.com/login/oauth/authorize?client_id=xxxxxxxxxx&state=xxxxxx&redirect_uri=http%3A%2F%2Flocalhost%3A9005&scope=read%3Auser%20repo%20public_repo
{% gist 078e5f77b04d0923330df0f7f1d013c2 %}
name: Deploy to Firebase Hosting on merge
'on':
  push:
    branches:
      - main
jobs:
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: npm ci && npm run build
      - uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: '${{ secrets.GITHUB_TOKEN }}'
          firebaseServiceAccount: '${{ secrets.FIREBASE_SERVICE_ACCOUNT_GITHUB_TEST_APP }}'
          channelId: live
          projectId: github-test-app
        env:
          FIREBASE_CLI_PREVIEWS: hostingchannels
```

All you need to do is to create the actions secret `FIREBASE_SERVICE_ACCOUNT_GITHUB_TEST_APP` and update it with the firebase-hosting service account ID (grab it from firebase [service account](https://console.cloud.google.com/projectselector2/iam-admin/serviceaccounts))

![Actions Secrets](https://github.com/superwalnut/mrkevin.wang/blob/master/images/github-project-secret.png)

