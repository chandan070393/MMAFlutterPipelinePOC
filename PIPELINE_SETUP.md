# Android Firebase Pipeline Setup

## Overview
This pipeline automatically builds and deploys your Flutter Android app to Firebase App Distribution when you push to main or develop branches.

## Prerequisites
1. Firebase project with App Distribution enabled
2. GitHub repository with Actions enabled
3. Firebase service account credentials

## Setup Instructions

### 1. Create Firebase Service Account

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Select your project: `mmaflutterpipelinegithub`
3. Go to Project Settings → Service Accounts
4. Click "Generate New Private Key"
5. Save the JSON file securely

### 2. Configure GitHub Secrets

Add these secrets to your GitHub repository (Settings → Secrets and variables → Actions):

#### FIREBASE_APP_ID
- Value: `1:27800772734:android:41ca99d4c256e6059af82f`
- This is your Android app ID from firebase.json

#### FIREBASE_SERVICE_ACCOUNT
- Copy the entire contents of the service account JSON file
- Paste it as the secret value

### 3. Setup Firebase App Distribution

1. Go to Firebase Console → App Distribution
2. Create a tester group named "testers" (or modify the workflow to use a different group)
3. Add testers' email addresses to the group

### 4. Optional: Setup App Signing (Recommended for Production)

For production releases, you should sign your app:

1. Generate a keystore:
```bash
keytool -genkey -v -keystore android/app/upload-keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

2. Create `android/key.properties`:
```properties
storePassword=<your-store-password>
keyPassword=<your-key-password>
keyAlias=upload
storeFile=upload-keystore.jks
```

3. Add to GitHub Secrets:
   - `KEYSTORE_BASE64`: Base64 encoded keystore file
   - `KEYSTORE_PASSWORD`: Your keystore password
   - `KEY_ALIAS`: Your key alias
   - `KEY_PASSWORD`: Your key password

4. Update the workflow to use these secrets for signing

## Workflow Triggers

The pipeline runs on:
- Push to `main` or `develop` branches
- Pull requests to `main`
- Manual trigger via GitHub Actions UI

## Build Outputs

The workflow produces:
- **APK**: For direct installation and Firebase App Distribution
- **App Bundle (AAB)**: For Google Play Store deployment

Both are available as downloadable artifacts in the GitHub Actions run.

## Testing the Pipeline

1. Commit and push your changes:
```bash
git add .
git commit -m "Add Android Firebase pipeline"
git push origin main
```

2. Go to GitHub Actions tab to monitor the workflow
3. Once complete, check Firebase App Distribution for the new build
4. Testers will receive an email notification

## Customization

### Change Tester Group
Edit `.github/workflows/android-firebase-deploy.yml`:
```yaml
groups: your-group-name
```

### Deploy App Bundle Instead of APK
Change the file path in the workflow:
```yaml
file: build/app/outputs/bundle/release/app-release.aab
```

### Add Build Variants
Modify the build commands:
```yaml
- name: Build APK (Staging)
  run: flutter build apk --release --flavor staging
```

## Troubleshooting

### Build Fails
- Check Flutter version compatibility
- Verify all dependencies are in pubspec.yaml
- Review build logs in GitHub Actions

### Firebase Deploy Fails
- Verify FIREBASE_APP_ID matches your app
- Check service account has App Distribution Admin role
- Ensure tester group exists in Firebase Console

### Signing Issues
- Verify keystore secrets are correctly encoded
- Check key.properties configuration
- Ensure signing config is properly set in build.gradle.kts
