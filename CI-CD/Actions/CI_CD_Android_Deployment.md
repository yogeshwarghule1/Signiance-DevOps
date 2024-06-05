
# CI/CD Implementation for Android Application Deployment on Google Play Store

This document outlines the step-by-step process to implement a CI/CD pipeline for an Android application to deploy it on the Google Play Store using GitHub Actions.

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Setup Steps](#setup-steps)
4. [CI/CD Pipeline Configuration](#cicd-pipeline-configuration)
5. [Detailed Steps and Configuration](#detailed-steps-and-configuration)
6. [Conclusion](#conclusion)

---

## 1. Introduction

Continuous Integration and Continuous Deployment (CI/CD) pipelines automate the process of building, testing, and deploying applications. This guide focuses on deploying an Android application to the Google Play Store using GitHub Actions for CI/CD.

## 2. Prerequisites

Before setting up the CI/CD pipeline, ensure the following prerequisites are met:
- GitHub repository for the Android application.
- Google Play Developer account.
- Service account with appropriate permissions in Google Play Console.
- Secrets configured in GitHub for secure information management.

## 3. Setup Steps

1. **Create a Service Account in Google Play Console**:
   - Log in to the Google Play Console.
   - Go to **Settings** > **API access**.
   - Create a new project if necessary.
   - Click **Create Service Account** and follow the prompts in the Google API Console.
   - Save the JSON key file.

2. **Configure Secrets in GitHub**:
   - Add the following secrets in your GitHub repository:
     - `SIGNING_KEY`: Base64 encoded signing key.
     - `ALIAS`: Alias for the key.
     - `KEY_STORE_PASSWORD`: Password for the keystore.
     - `KEY_PASSWORD`: Password for the key.
     - `SERVICE_ACCOUNT_JSON`: Content of the service account JSON key file.

## 4. CI/CD Pipeline Configuration

### GitHub Actions Workflow File

Create a `.yml` file in the `.github/workflows` directory of your repository. Name it `android-ci-cd.yml` and add the following content:

\`\`\`yaml
name: Deploy App CI CD

on:
  push:
    branches: [ master ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v2

      - name: Set up JDK 1.8
        uses: actions/setup-java@v1
        with:
          java-version: 1.8

      - name: Grant rights
        run: chmod +x build.gradle

      - name: Build with Gradle
        id: build
        run: ./gradlew build

      - name: Build Release AAB
        id: buildRelease
        run: ./gradlew bundleRelease

      - name: Sign AAB
        id: sign
        uses: r0adkll/sign-android-release@v1
        with:
          releaseDirectory: app/build/outputs/bundle/release
          signingKeyBase64: ${{ secrets.SIGNING_KEY }}
          alias: ${{ secrets.ALIAS }}
          keyStorePassword: ${{ secrets.KEY_STORE_PASSWORD }}
          keyPassword: ${{ secrets.KEY_PASSWORD }}

      - name: Create service_account.json
        id: createServiceAccount
        run: echo '${{ secrets.SERVICE_ACCOUNT_JSON }}' > service_account.json

      - name: Deploy to Play Store (BETA)
        id: deploy
        uses: r0adkll/upload-google-play@v1
        with:
          serviceAccountJson: service_account.json
          packageName: com.package
          releaseFile: app/build/outputs/bundle/release/app-release.aab
          track: beta
\`\`\`

## 5. Detailed Steps and Configuration

### 5.1 Checkout

**Description**: This step retrieves the source code from your GitHub repository.
\`\`\`yaml
- name: Checkout
  uses: actions/checkout@v2
\`\`\`

### 5.2 Set up JDK 1.8

**Description**: Installs Java Development Kit (JDK) version 1.8.
\`\`\`yaml
- name: Set up JDK 1.8
  uses: actions/setup-java@v1
  with:
    java-version: 1.8
\`\`\`

### 5.3 Grant Rights

**Description**: Grants execute permissions to the `build.gradle` file.
\`\`\`yaml
- name: Grant rights
  run: chmod +x build.gradle
\`\`\`

### 5.4 Build with Gradle

**Description**: Builds the Android application using Gradle.
\`\`\`yaml
- name: Build with Gradle
  id: build
  run: ./gradlew build
\`\`\`

### 5.5 Build Release AAB

**Description**: Creates an Android App Bundle (AAB).
\`\`\`yaml
- name: Build Release AAB
  id: buildRelease
  run: ./gradlew bundleRelease
\`\`\`

### 5.6 Sign AAB

**Description**: Digitally signs the AAB.
\`\`\`yaml
- name: Sign AAB
  id: sign
  uses: r0adkll/sign-android-release@v1
  with:
    releaseDirectory: app/build/outputs/bundle/release
    signingKeyBase64: ${{ secrets.SIGNING_KEY }}
    alias: ${{ secrets.ALIAS }}
    keyStorePassword: ${{ secrets.KEY_STORE_PASSWORD }}
    keyPassword: ${{ secrets.KEY_PASSWORD }}
\`\`\`

### 5.7 Create Service Account JSON

**Description**: Creates the `service_account.json` file from the secret.
\`\`\`yaml
- name: Create service_account.json
  id: createServiceAccount
  run: echo '${{ secrets.SERVICE_ACCOUNT_JSON }}' > service_account.json
\`\`\`

### 5.8 Deploy to Play Store

**Description**: Uploads the signed AAB to the Google Play Console.
\`\`\`yaml
- name: Deploy to Play Store (BETA)
  id: deploy
  uses: r0adkll/upload-google-play@v1
  with:
    serviceAccountJson: service_account.json
    packageName: com.package
    releaseFile: app/build/outputs/bundle/release/app-release.aab
    track: beta
\`\`\`

## 6. Conclusion

Implementing a CI/CD pipeline for your Android application using GitHub Actions automates the entire process of building, testing, and deploying your app to the Google Play Store. This setup ensures a consistent, efficient, and secure deployment process, allowing your development team to focus on creating great features and improving the app.
