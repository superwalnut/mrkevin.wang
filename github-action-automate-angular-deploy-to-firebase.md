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

``` bash
npm install -g @angular/cli
```

### Create an angular app if you haven't built one

``` bash
ng new test-app
```
run ng serve to test it locally, once your app has no error and runs successfully, let's go and publish it to firebase.

## Part 2. Setup firebase

- Firstly, of course you need to create a firebase account and a firebase project in [firebase console](https://console.firebase.google.com/).

- Then install the firebase tools.

``` bash
npm install -g firebase-tools
```

- Login in to firebase using its cli

``` bash
firebase login
```

- That allows you use firebase cli to initialize a firebase project.

``` bash
firebase init
```

- You will see a few options that you can use your space bar to select features and hit enter to continue

```
Database: Deploy Firebase Real-time Database Rules
Firestore: Deploy rules and create indexes for Firestore
Functions: Configure and deploy Cloud Functions
Hosting: Configure and deploy Firebase Hosting sites
Storage: Deploy Cloud Storage security rules
```

We are only using hosting, select it and hit enter. Firebase CLI will ask you a few questions, they are all pretty straight forward. 

I want to mention this question, `Configure as a single-page app (rewrite all URLs to /index.html)? (y/N)`, as most of angular apps are created as Single page app, so I would choose `yes`. Of course if you have different routing strategies, it might be a different selection for you.

### Setup github actions

Also it provides an option if you want it to create a github action file for you automatically, how sweet is that, save us plenty of time.

```
 Set up automatic builds and deploys with GitHub?
```

select yes and let it does the rest of the job. In the end it will open a link and ask you `authorize firebase cli access`. The link looks similar to

``` bash
https://github.com/login/oauth/authorize?client_id=xxxxxxxxxx&state=xxxxxx&redirect_uri=http%3A%2F%2Flocalhost%3A9005&scope=read%3Auser%20repo%20public_repo
```

Once that is completed, it creates workflows yml file for github actions.

```
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

## Part 3. Miscellaneous

### Fix output folder

The firebase cli generates a firebase.json to provide instructions of deployment. You may notice it sets `public` folder to be `public`, so when we build the angular app, we need to build into `public` folder (instead of default `dist` folder).
I am updating the `npm build` command of `package.json` to

``` bash
   ng build --prod --output-path=public
```

### Manual deploy

You can also manual deploy it via firebase cli, by running

``` bash
    firebase deploy
```

## Part 4. Finally

Please find the sample application with firebase deploy setup and github workflows action file at this link

https://github.com/superwalnut/github-actions-angular-firebase-sample

